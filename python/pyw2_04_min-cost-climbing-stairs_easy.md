## challenge: Min Cost Climbing Stairs
tags: dp, array
track: python
lang: python
difficulty: easy

You are given a list `cost` where `cost[i]` is the price of stepping on stair `i`. Once you pay, you may climb one or two stairs. You start from stair `0` or stair `1` for free (you pay when you step off it). Return the minimum total cost to reach the top — one past the last stair.

Constraints: `2 <= len(cost) <= 1000`, `0 <= cost[i] <= 999`.

Example: `cost = [10, 15, 20]` → `15` (start on stair 1, pay 15, jump two to the top).

hint: Let `dp[i]` be the cheapest way to *leave* stair `i`. How do you arrive at stair `i` in the first place?
hint: `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`, and the answer is `min(dp[n-1], dp[n-2])`.
hint: You only ever look two steps back — two rolling variables replace the whole array.

```python
# starter
def min_cost_climbing_stairs(cost):
    ...
```

```python
def min_cost_climbing_stairs(cost):
    a, b = 0, 0
    for c in cost:
        a, b = b, min(a, b) + c
    return min(a, b)
```

```python
# harness
#__USER__
def _check():
    assert min_cost_climbing_stairs([10, 15, 20]) == 15
    assert min_cost_climbing_stairs([1, 100, 1, 1, 1, 100, 1, 1, 100, 1]) == 6
    assert min_cost_climbing_stairs([0, 0]) == 0
    assert min_cost_climbing_stairs([5, 10]) == 5
    assert min_cost_climbing_stairs([1, 2]) == 1
    assert min_cost_climbing_stairs([0, 1, 2, 2]) == 2
    print("PASS")

_check()
```

**Editorial:** Bottom-up DP where `dp[i]` is the cost paid up to and including leaving stair `i`: `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`. Because you may start on stair 0 or 1, the two seeds are 0, and because you may finish from either of the last two stairs, the answer is `min(dp[n-1], dp[n-2])`. Rolling two variables gives O(n) time, O(1) space.
