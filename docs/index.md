---
icon: lucide/terminal-square
hide:
  - toc
---

# Production code is awful. 
**Stop adding frameworks. Start bending the runtime.**

Real codebases are graveyards of explicit state: retry loops, max-worker configs, and Saga patterns. We build orthogonal Python primitives that rely purely on native interpreter physics (OS forks, bytecode pointers, exception bubbling) to achieve the impossible with zero boilerplate.

=== "Parallelize without Pickling"

    **`scissiparity`**: Clones the interpreter natively via OS-level Copy-On-Write. 0 bytes duplicated. 0 IPC serialization errors.

    ```python hl_lines="6"
    from scissiparity import fission

    @fission
    def process_users(user_ids: list[str]):
        for uid in user_ids:
            state = MASSIVE_UNPICKLABLE_DICT[uid] # Native OS memory read
            yield heavy_work(state)               # Magically pipes back across N processes
    ```

=== "Fix N+1 without Queues"

    **`concresce`**: Coalesces concurrent function calls inline using the native async event loop tick.

    ```python hl_lines="5"
    from concresce import batch, collect

    @batch
    async def fetch_score(user_id):
        batch_ids = await collect(user_id) # Execution suspends. Inputs pool naturally.

        # Only ONE execution path resumes to make 1 network call.
        return await db.bulk_fetch(batch_ids) 
    ```

=== "Safe Side-Effects without Sagas"

    **`epilogue`**: Defers side-effects orthogonally. If the function crashes, the queue is mathematically annihilated.

    ```python hl_lines="5"
    import epilogue

    @epilogue.atomic
    def checkout(user, cart):
        epilogue.defer(stripe.charge, cart.total) # Intent queued. NO network call made.
        
        db.save_order(user, cart) # If this crashes, Stripe is safely aborted.
        return True
    ```

---

<div class="grid cards" markdown>

-   :lucide-cpu: __Concurrency & Batching__

    ---

    Destroy the GIL, eliminate N+1 queries natively, and branch execution realities dynamically.

    **`scissiparity`** · **`concresce`** · **`bifurcate`**

    [:lucide-arrow-right: Explore Concurrency](concurrency.md)

-   :lucide-lock: __State & Locks__

    ---

    Acquire absolute control over deferred side-effects and distributed locking mid-execution.

    **`epilogue`** · **`caesura`**[:lucide-arrow-right: Explore State](state.md)

-   :lucide-beaker: __Testing & Caching__

    ---

    Safely shadow-test legacy refactors in production, and memoize using real-time state.

    **`bilocate`** · **`belljar`**[:lucide-arrow-right: Explore Testing](testing.md)

</div>

---

```bash title="Install what you need, or grab the suite"
uv add scissiparity concresce bifurcate epilogue caesura bilocate belljar
```
