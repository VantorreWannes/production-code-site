---
icon: lucide/beaker
---

# Testing & Caching

Testing and caching are usually solved by external infrastructure (API gateways, Redis clusters). We solve them purely through derived identity and native python evaluation (`__eq__` and `__code__`).

---

## bilocate 🪞

**Transparent shadow testing for legacy refactors.**

Replacing spaghetti V1 logic with clean V2 logic is terrifying. `bilocate` allows you to run V2 in the background natively. If the outputs differ, it emits a standard `RuntimeWarning`.

```python title="bilocate" hl_lines="6"
import bilocate

def clean_v2_logic(payload):
    return new_pydantic_math(payload)

@bilocate.mimic(clean_v2_logic) # (1)!
def spaghetti_v1_logic(payload):
    return legacy_dict_math(payload)

# User gets V1 response instantly.
result = spaghetti_v1_logic({"id": 123})
```

1. **Exceptions are Values:** If V1 succeeds but V2 crashes, `bilocate` evaluates the exception natively and logs the mismatch without disrupting the user's stack.

---

## belljar 🫙

**Mid-execution memoization for dynamic state.**

`functools.lru_cache` fails on stateful objects (like file cursors) because it checks the cache _before_ the function runs. `belljar` builds the identity hash mid-execution.

```python title="belljar" hl_lines="6 9 12"
import belljar

@belljar.store # (1)!
def process_chunk(file_handle):
    # 1. Add the file's exact runtime cursor position to the identity hash
    belljar.include(file_handle)

    # 2. If we've processed from this exact state before, short-circuit.
    belljar.check() # (2)!

    return file_handle.read(6)
```

1. **Auto-Invalidation:** `belljar` hashes the native `__code__.co_code`. If you edit the script, the cache naturally invalidates.
2. **Exceptional Control Flow:** `check()` raises a hidden exception to instantly teleport execution out of the function, returning the cached disk payload without `if/else` boilerplate.
