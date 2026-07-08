## challenge: Memoize with lru_cache
tags: functools, memoization
track: python
lang: python
difficulty: hard

Implement `fib(n)` returning the n-th Fibonacci number (`fib(0)=0`, `fib(1)=1`) using the naive recursion `fib(n) = fib(n-1) + fib(n-2)` — but make it fast by caching results so each argument is computed at most once.

Constraints: `n >= 0`. The graded solution must expose cache statistics (i.e. the memoized function's `cache_info()` / `cache_clear()`).

Example: `fib(10)` → `55`; recomputing `fib(10)` afterward is a pure cache hit.

hint: The naive recursion recomputes the same subproblems exponentially — memoization collapses that to linear.
hint: `functools.lru_cache` transparently caches a function's return value keyed by its arguments.
hint: Decorate the recursive function: `@lru_cache(maxsize=None)` above `def fib(n): ...`.

```python
# starter
def fib(n):
    ...
```

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)
```

```python
# harness
#__USER__
def _check():
    assert fib(0) == 0
    assert fib(1) == 1
    assert fib(10) == 55
    assert fib(30) == 832040
    fib.cache_clear()
    assert fib(25) == 75025
    stats = fib.cache_info()
    # overlapping subproblems are reused, so there must be cache hits
    assert stats.hits > 0
    # each distinct argument 0..25 is computed exactly once
    assert stats.misses == 26
    # a repeat call adds only hits, no new misses
    misses_before = fib.cache_info().misses
    assert fib(25) == 75025
    assert fib.cache_info().misses == misses_before
    print("PASS")

_check()
```

**Editorial:** `functools.lru_cache` wraps the function in a memoizing cache keyed on its arguments, turning the exponential naive recursion into linear time — each of `fib(0)..fib(n)` is computed once and every other reference is a hit. `maxsize=None` gives an unbounded cache (no eviction), and the wrapper exposes `cache_info()`/`cache_clear()` for introspection. One decorator replaces a hand-rolled memo dict and its lookup/store boilerplate.
