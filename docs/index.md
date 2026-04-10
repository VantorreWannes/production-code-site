---
icon: lucide/home
---

# 🛠️ Pragmatic Python Tools

Welcome to the execution manipulation suite.

We didn't build these libraries to be academic. We built them because standard Python often forces us into writing massive architectural boilerplate to solve simple problems—like setting up Redis queues just to batch a few database calls, or wrestling with `PicklingError`s.

These 7 lightweight libraries solve complex infrastructure headaches natively, right at the Python function boundary.

### Quick Start

Identify a bottleneck in your app, pick the exact tool below that solves it, and delete your boilerplate code.

| Library             | The Real-World Problem It Solves                                                   | Read More                                          |
| :------------------ | :--------------------------------------------------------------------------------- | :------------------------------------------------- |
| **🫙 belljar**      | Caching fails on mutable state (like open files or DB cursors).                    | [Mid-execution caching](./belljar.md)              |
| **🌿 bifurcate**    | A/B tests or sweeps force you to rewrite logic into ugly loops.                    | [Inline branching](./bifurcate.md)                 |
| **🪞 bilocate**     | Shadow-testing legacy code usually requires an API gateway.                        | [Legacy refactoring](./bilocate.md)                |
| **📜 caesura**      | Context managers force you to lock resources _before_ you know what to lock.       | [Yield-based locking](./caesura.md)                |
| **💧 concresce**    | FastAPI/GraphQL N+1 queries usually require heavy Celery workers.                  | [Inline micro-batching](./concresce.md)            |
| **🎭 epilogue**     | If a DB insert crashes _after_ charging a credit card, you're in trouble.          | [Deferred side-effects](./epilogue.md)             |
| **🦠 scissiparity** | Multiprocessing copies massive memory payloads and crashes on unpicklable objects. | [Copy-on-write multiprocessing](./scissiparity.md) |

Explore the tools below to see exactly _why_ they beat standard Python patterns.
