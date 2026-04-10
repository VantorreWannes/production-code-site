---
icon: lucide/droplets
---

# 💧 Concresce: Inline Micro-Batching

Imagine you have a FastAPI endpoint. Under high load, 50 requests come in at the exact same millisecond. Instead of making 50 individual database queries (the classic N+1 problem), **concresce** transparently merges them in the background.

No heavy Celery queues. Just pure `asyncio`.

## The Code

```python hl_lines="4 8"
import asyncio
from concresce import batch, collect

@batch
async def get_user(user_id):
    # 1. Execution pauses. 
    # All concurrent requests in this loop tick pool their user_ids here.
    batch_ids = await collect(user_id) # (1)!

    # 2. Only ONE request (the leader) resumes. The others safely suspend.
    users_dict = await db.fetch_many(batch_ids)

    # 3. concresce routes the correct dictionary value back to the followers.
    return users_dict 
```

1. `await collect()` yields exactly once to the event loop. If traffic is low, it executes instantly. If traffic spikes, it naturally pools massive batches.

!!! warning "What if the database crashes?"
    If the heavy processing throws an error, `concresce` intercepts it and safely replicates the exception to **all** suspended followers. Your endpoints never hang, and users get proper 500 errors.