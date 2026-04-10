---
icon: lucide/split-square-horizontal
---

# 🪞 Bilocate: Shadow Testing Legacy Refactors

Shadow testing a legacy refactor usually requires standing up API gateways, dual-writing Kafka queues, and using heavy JSON-diffing engines just to see if your new code matches the old code.

**bilocate** does it natively in the background, entirely within Python.

## The Code

```python hl_lines="6"
import bilocate

def clean_v2(payload):
    return PydanticOrder(**payload).dict()

@bilocate.mimic(clean_v2) # (1)!
def spaghetti_v1(payload):
    return messy_legacy_parser(payload)

# The user gets the V1 response instantly. Zero latency added.
result = spaghetti_v1({"user_id": 123})
```

1. The decorator intercepts the arguments and automatically pipes them to V2 in a background fire-and-forget task.

!!! success "Flawless Error Diffing"
    If V1 succeeds but V2 crashes or returns a different result, it does **not** blow up the user's stack. `bilocate` cleanly catches the discrepancy and reports it via a standard Python `RuntimeWarning`.

You don't need custom telemetry SDKs. You can route these warnings straight to your standard logger by just calling `logging.captureWarnings(True)` when your app starts up!