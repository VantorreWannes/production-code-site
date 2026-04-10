---
icon: lucide/lock
---

# State & Locks

Explicit resource hoarding and manual rollback logic are the leading causes of production rot. We rely on native `contextvars` and tuple-sorting to keep side-effects mathematically constrained.

---

## epilogue 📜

**Strict execution boundaries for deferred side-effects.**

You want to write to a DB and charge a Stripe card. If Stripe fails, the DB shouldn't commit. If the DB fails, Stripe shouldn't charge.

`epilogue` defers execution to the absolute end of the function. If the function crashes at any point, the queue is mathematically annihilated. **Zero side effects occur.**

```python title="epilogue" hl_lines="5"
import epilogue

@epilogue.atomic
def checkout(user, cart):
    epilogue.defer(stripe.charge, cart.total) # (1)!

    db.save_order(user, cart) # (2)!
    return True
```

1. **Context-Safe:** The queue lives in native `contextvars`. It does not bleed across threads or async loops.
2. **Exception Bubbling:** If `save_order` crashes, Python native exception bubbling destroys the local context. The deferred Stripe charge simply vanishes.

---

## caesura 🔓

**Mid-execution distributed locking with zero boilerplate.**

Standard locking wraps the entire function, locking resources before you even know if you need them. `caesura` obeys the Rule of Extreme Locality: lock exactly where the blocking I/O happens.

```python title="caesura" hl_lines="9-12"
import caesura

@caesura.serialize
def transfer_funds(sender_id, receiver_id):
    # Do heavy, lock-free work first
    parsed = heavy_parse(sender_id, receiver_id)

    # Yield hands the locks to the decorator mid-execution.
    yield (
        redis.lock(parsed.sender_id),
        redis.lock(parsed.receiver_id)
    ) # (1)!

    db.execute_transfer(parsed.sender_id, parsed.receiver_id) # (2)!
```

1. **Deadlocks Eradicated:** The tuple is intercepted and lexicographically sorted by identifier before acquisition. A Dining Philosophers deadlock is mathematically impossible.
2. **Native Teardown:** When the function returns (or crashes), Python's native `ExitStack` safely unwinds and releases the locks.
