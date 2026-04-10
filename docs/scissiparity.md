---
icon: lucide/cpu
---

# 🦠 Scissiparity: Copy-on-Write Multiprocessing

Python's standard `multiprocessing.Pool` tries to serialize (pickle) all inputs across pipes. If you have a 10GB machine learning model in memory, your program tries to copy it 8 times and instantly crashes from Out-Of-Memory errors.

**scissiparity** solves this by physically forking the Python interpreter. Child processes read the parent's memory instantly via the Operating System's Copy-on-Write. Inputs are free.

## The Code

=== "Standard Pool (Crashes)"

    ```python
    from multiprocessing import Pool

    MASSIVE_MODEL = load_10gb()

    def predict(user_id):
        # Pool tries to pickle MASSIVE_MODEL to send to workers. OOM Crash!
        return MASSIVE_MODEL.predict(user_id)

    with Pool() as p:
        results = p.map(predict,["u1", "u2"])
    ```

=== "With Scissiparity (Zero-copy)"

    ```python hl_lines="8 11"
    from scissiparity import fission

    MASSIVE_MODEL = load_10gb()

    @fission
    def process_users(user_ids: list[str]):
        for uid in user_ids:
            # Downward flow is FREE via native Copy-On-Write
            pred = MASSIVE_MODEL.predict(uid) # (1)!

            # Upward flow is serialized back to parent natively
            yield pred

    # Fissions automatically based on os.cpu_count()
    results = list(process_users(["u1", "u2"]))
    ```

    1.  This allows you to safely parallel-process objects that are completely unpicklable, like open database connections or massive Pandas dataframes.
