## challenge: Climbing Stairs
tags: dynamic-programming, math, memoization
track: python
lang: python
difficulty: easy

You are climbing a staircase with `n` steps. Each move you may climb either 1 or 2 steps. Return the number of distinct ways to reach the top.

Constraints: `1 <= n <= 45`.

Example: `n = 3` → `3` (the ways are `1+1+1`, `1+2`, and `2+1`).

hint: The number of ways to reach step `n` is the sum of the ways to reach `n-1` and `n-2`.
hint: That recurrence is exactly the Fibonacci sequence.
hint: Keep only the last two values instead of a full table — O(1) space.

```python
# starter
def climb_stairs(n):
    ...
```

```python
def climb_stairs(n):
    a, b = 1, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

```python
# harness
#__USER__
def _check():
    assert climb_stairs(1) == 1
    assert climb_stairs(2) == 2
    assert climb_stairs(3) == 3
    assert climb_stairs(5) == 8
    assert climb_stairs(45) == 1836311903
    print("PASS")

_check()
```

**Editorial:** To reach step `n`, your last move came from step `n-1` (a single step) or `n-2` (a double step), so `ways(n) = ways(n-1) + ways(n-2)`. That is the Fibonacci recurrence; rolling two variables forward gives O(n) time and O(1) space.
