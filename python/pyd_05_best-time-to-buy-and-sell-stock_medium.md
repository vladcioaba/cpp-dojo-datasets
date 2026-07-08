## challenge: Best Time to Buy and Sell Stock
tags: array, dynamic-programming
track: python
lang: python
difficulty: medium

Given a list `prices` where `prices[i]` is the price of a stock on day `i`, choose one day to buy and a later day to sell to maximize profit. Return the maximum profit, or `0` if no profitable trade exists.

Constraints: `0 <= len(prices) <= 10^5`, `0 <= prices[i] <= 10^4`. You must buy before you sell.

Example: `prices = [7, 1, 5, 3, 6, 4]` → `5` (buy at 1, sell at 6). `prices = [7, 6, 4, 3, 1]` → `0`.

hint: For each selling day, the best buy is the cheapest price seen so far.
hint: Track the running minimum price in a single left-to-right pass.
hint: At each day, update the best profit as `price - min_so_far`, then update the minimum.

```python
# starter
def max_profit(prices):
    ...
```

```python
def max_profit(prices):
    min_price = float('inf')
    best = 0
    for p in prices:
        if p < min_price:
            min_price = p
        elif p - min_price > best:
            best = p - min_price
    return best
```

```python
# harness
#__USER__
def _check():
    assert max_profit([7, 1, 5, 3, 6, 4]) == 5
    assert max_profit([7, 6, 4, 3, 1]) == 0
    assert max_profit([1, 2, 3, 4, 5]) == 4
    assert max_profit([]) == 0
    assert max_profit([3]) == 0
    assert max_profit([2, 4, 1]) == 2
    print("PASS")

_check()
```

**Editorial:** As you scan the days, remember the lowest price seen up to the current day — that is the best price you could have bought at for any sell on today. The answer is the maximum of `today_price - min_so_far` across all days, never letting profit drop below `0`. One pass, O(n) time and O(1) space, versus checking every buy/sell pair in O(n²).
