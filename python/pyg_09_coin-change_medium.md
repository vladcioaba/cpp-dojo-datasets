## challenge: Coin Change
tags: dynamic-programming, bfs
track: python
lang: python
difficulty: medium

Given a list of `coins` of distinct denominations and an integer `amount`, return the fewest coins needed to make up that amount. You may use each denomination any number of times. If the amount cannot be made, return `-1`.

Constraints: `1 <= len(coins) <= 12`, `1 <= coins[i] <= 2^31 - 1`, `0 <= amount <= 10^4`.

Example: `coins = [1, 2, 5], amount = 11` → `3` (`5 + 5 + 1`).

hint: Build up the answer for every sub-amount from `0` to `amount`.
hint: `dp[a] = 1 + min(dp[a - c])` over every coin `c` that fits in `a`.
hint: Seed `dp[0] = 0` and treat any amount that stays unreachable as `-1`.

```python
# starter
def coin_change(coins, amount):
    ...
```

```python
def coin_change(coins, amount):
    INF = amount + 1
    dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return dp[amount] if dp[amount] <= amount else -1
```

```python
# harness
#__USER__
def _check():
    assert coin_change([1, 2, 5], 11) == 3
    assert coin_change([2], 3) == -1
    assert coin_change([1], 0) == 0
    assert coin_change([1, 2, 5], 100) == 20
    assert coin_change([2, 5, 10, 1], 27) == 4
    assert coin_change([186, 419, 83, 408], 6249) == 20
    print("PASS")

_check()
```

**Editorial:** Bottom-up DP over amounts. `dp[a]` is the fewest coins summing to `a`; for each amount try every coin that fits and take `1 + dp[a - c]`. Initialise `dp[0] = 0` and a sentinel `amount + 1` for unreachable amounts, which maps to `-1` at the end. O(amount · len(coins)) time, O(amount) space.
