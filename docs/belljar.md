---
icon: lucide/box
---

# 🫙 Belljar: Mid-Execution Caching

Standard caching is great for pure math. But it fails when your identity relies on mutable state—like reading a file or a database cursor. Standard tools check the cache _before_ the function runs, meaning they miss internal state changes and return stale data.

**belljar** fixes this by letting you intercept the execution _while_ it runs.

## The Code

Notice how we pause exactly where the state matters.

=== "Standard caching (Fails)"

    ```python
    from functools import lru_cache
    import io

    f = io.StringIO("line1\nline2\nline3")

    @lru_cache
    def read_next(file_obj):
        return file_obj.readline()

    print(read_next(f)) # "line1\n"
    print(read_next(f)) # "line1\n" ==(Stale!)==
    ```

=== "With belljar (Succeeds)"

    ```python hl_lines="8 11"
    import belljar
    import io

    f = io.StringIO("line1\nline2\nline3")

    @belljar.store
    def read_next(file_obj):
        # Tell belljar the EXACT state of the file right now
        belljar.include(file_obj.tell()) # (1)!

        # Intercept! Stop running if we've seen this state before
        belljar.check()

        return file_obj.readline()

    print(read_next(f)) # "line1\n"
    print(read_next(f)) # "line2\n" ==(Correct!)==
    ```

    1.  This allows us to track dynamic properties like `.tell()` offsets instead of just static memory addresses.

Instead of rewriting your architecture to pass around explicit string offsets or building custom caching dictionaries, you just track the state that matters inside the function itself.

_(Plus, if you rewrite the function later, the cache automatically busts itself by hashing your source code!)_
