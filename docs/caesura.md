---
icon: lucide/lock
---

# 📜 Caesura: Yield-Based Distributed Locks

Context managers (like `with redis.lock():`) are rigid. They force you to lock resources *before* you've parsed the payload to see what you actually need. If you try to lock mid-function, you risk leaving hanging locks if the database crashes halfway through.

**caesura** allows you to safely lock mid-execution by simply `yield`ing the lock.

## The Code

=== "Context Managers (Rigid)"

    ```python
    def transfer(sender, receiver):
        # We have to nest manually
        # HIGH RISK of Deadlock if workers lock in different orders!
        with redis.lock(sender):
            with redis.lock(receiver):
                db.transfer(sender, receiver)
    ```

=== "With Caesura (Flexible & Safe)"

    ```python hl_lines="6"
    import caesura

    @caesura.serialize
    def transfer(sender, receiver):
        # Hand the locks to the decorator. 
        # They are ==sorted== automatically to prevent deadlocks.
        yield (redis.lock(sender), redis.lock(receiver)) # (1)!
        
        db.transfer(sender, receiver)
    ```

    1. If the function crashes after yielding, the decorator still safely releases the locks. No hanging resources.

The Dining Philosophers deadlock happens when workers acquire resources in different orders. `caesura` solves this mathematically by sorting tuples of locks natively before acquiring them!