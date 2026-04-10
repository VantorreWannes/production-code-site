---
icon: lucide/git-branch
---

# 🌿 Bifurcate: Inline Execution Branching

If you want to run an A/B test or a hyperparameter sweep, standard Python forces you to extract your logic, build a `for` loop, and manage lists. It pollutes your clean business logic.

**bifurcate** lets you physically split the execution universe *inside* the function.

## The Code

Instead of rewriting your single-item math into batch-processing arrays, write it as if it runs exactly once.

=== "The loop way (Ugly)"

    ```python
    def get_price(uid, discount):
        base = db.get(uid)
        return base * (1 - discount)

    # Caller has to manage loops and aggregations
    results = []
    for d in[0.0, 0.10, 0.20]:
        results.append(get_price("u1", d))
    ```

=== "With bifurcate (Clean)"

    ```python hl_lines="6"
    import bifurcate

    @bifurcate.collect
    def get_price(uid):
        base = db.get(uid)
        
        # Fork the function right here
        discount = bifurcate.branch([0.0, 0.1, 0.2]) # (1)!
        
        return base * (1 - discount)

    # Natively returns the full array of permutations
    results = get_price("u1")
    ```

1.  Because `bifurcate` reads the internal Python bytecode, you can safely drop `branch()` inside deep `if` statements or nested helper functions without breaking anything.