---
icon: lucide/scroll-text
---

# 🎭 Epilogue: Deferred Side-Effects

Here is a classic failure mode: A user buys an item. Your code hits the network to charge their Stripe account, then tries to insert the order into your database. The database crashes. **Your user's card was charged, but they have no order.** 

**epilogue** fixes this by queuing side-effects and ensuring they *only* execute if the core function returns successfully.

## The Code

=== "Dangerous"

    ```python
    def checkout(user, cart):
        stripe.charge(user) # Network call happens immediately!
        
        db.save(user, cart) # If this crashes...
        email.send(user)    # ...we are in a bad state.
        
        return True
    ```

=== "With epilogue (Safe)"

    ```python hl_lines="5 6"
    import epilogue

    @epilogue.atomic
    def checkout(user, cart):
        # Hit NO network. Just register intent in memory.
        epilogue.defer(stripe.charge, user) # (1)!
        epilogue.defer(email.send, user)
        
        db.save(user, cart) # Crash-prone DB logic
        
        return True # (2)!
    ```

    1. Adds the intent to a lightweight in-memory queue. Topologically safe across concurrent threads!
    2. Once the function successfully returns, the deferred actions are safely executed in order. 

If the database crashes, the `epilogue` queue is silently destroyed. Zero network calls are made, guaranteeing an all-or-nothing transaction without complex rollback logic.