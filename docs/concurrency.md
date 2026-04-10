---
icon: lucide/cpu
---

# Concurrency & Batching

Standard concurrency forces you to orchestrate the infrastructure: managing worker pools, chunking iterables, and serializing data across process boundaries. 

We push orchestration down into the interpreter. You write sequential logic; the OS and event loop scale it.

---

## scissiparity 🦠

**Asymmetric serialization for GIL-free parallel map-reduce.**

If you pass a 2GB dataframe or a database connection to `multiprocessing.Pool`, Python duplicates the memory via IPC queues and immediately crashes with a `PicklingError`. `scissiparity` uses `os.fork()` to clone the memory space natively. Inputs are read for free via OS-level Copy-On-Write. Only the results are serialized.

```python title="scissiparity" hl_lines="5"
from scissiparity import fission

MASSIVE_UNPICKLABLE_DICT = {"user_1": db_conn, "user_2": cache_conn}

@fission # (1)!
def process_users(user_ids: list[str]):
    for uid in user_ids:
        # ⬇️ Native OS read. 0 bytes duplicated. Zero pickling errors.
        state = MASSIVE_UNPICKLABLE_DICT[uid]
        
        # ⬆️ Only the result is serialized and multiplexed back.
        yield compute(state) 

results = list(process_users(["user_1", "user_2", "user_3"]))
```

1. **Zero Configuration:** No `workers=8` parameter. The decorator automatically fissions based on `os.sched_getaffinity()` or `os.cpu_count()`.

---

## concresce 💧

**Transparent temporal coalescing and inline micro-batching.**

Solving the N+1 problem usually requires a message broker. `concresce` coalesces function calls inline natively using the async event loop's microtask queue. 

No timeouts. No `max_batch_size`. The batch window is the exact native tick of the interpreter.

```python title="concresce" hl_lines="6 9"
import asyncio
from concresce import batch, collect

@batch
async def fetch_score(user_id):
    batch_ids = await collect(user_id) # (1)!

    # Only ONE execution path resumes. The rest remain safely suspended.
    return await db.bulk_fetch(batch_ids) # (2)!

# Fire 5 requests instantly
results = await asyncio.gather(*[fetch_score(i) for i in range(5)])
# Output:[100, 250, 190, 300, 120]
```

1. **The Barrier:** `concresce` yields exactly once (`await asyncio.sleep(0)`) to the loop. Concurrent calls pool their inputs here.
2. **Native Routing:** The leader returns a dictionary (`{id: result}`). The decorator maps the results back to the exact followers using the key.

---

## bifurcate 🔱

**Emergent parallelism and execution branching.**

Standard A/B tests or hyperparameter sweeps pollute your domain logic with `for` loops. `bifurcate` achieves the List Monad in Python by fracturing the execution graph from *inside* the function using the bytecode instruction pointer (`f_lasti`).

```python title="bifurcate" hl_lines="7"
import bifurcate

@bifurcate.collect
def calculate_pricing(user_id):
    base_price = db.get_price(user_id)

    discount = bifurcate.branch([0.0, 0.10, 0.20]) # (1)!

    return {"discount": discount, "price": base_price * (1 - discount)}

# The decorator automatically aggregates the permutations
results = calculate_pricing("user_123")
```

1. **The Fork:** The function pauses, saves its frame state, and replays the execution from this exact bytecode line for every item in the list.
