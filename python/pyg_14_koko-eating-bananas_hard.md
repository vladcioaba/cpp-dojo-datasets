## challenge: Koko Eating Bananas
tags: binary-search, array
track: python
lang: python
difficulty: hard

Koko has `len(piles)` piles of bananas and `h` hours before the guards return. At a chosen integer eating speed `k` (bananas per hour) she eats from one pile each hour; if a pile has fewer than `k` bananas left she finishes it and waits out the hour. Return the minimum `k` that lets her eat every banana within `h` hours.

Constraints: `1 <= len(piles) <= 10^4`, `len(piles) <= h <= 10^9`, `1 <= piles[i] <= 10^9`.

Example: `piles = [3, 6, 7, 11], h = 8` → `4`.

hint: The answer lies between `1` and `max(piles)` — binary search that range.
hint: For a candidate speed `k`, a pile of `p` bananas takes `ceil(p / k)` hours.
hint: Faster speeds never need more hours, so the feasibility test is monotonic — perfect for binary search.

```python
# starter
def min_eating_speed(piles, h):
    ...
```

```python
def min_eating_speed(piles, h):
    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        hours = sum((p + mid - 1) // mid for p in piles)
        if hours <= h:
            hi = mid
        else:
            lo = mid + 1
    return lo
```

```python
# harness
#__USER__
def _check():
    assert min_eating_speed([3, 6, 7, 11], 8) == 4
    assert min_eating_speed([30, 11, 23, 4, 20], 5) == 30
    assert min_eating_speed([30, 11, 23, 4, 20], 6) == 23
    assert min_eating_speed([312884470], 968709470) == 1
    assert min_eating_speed([1000000000], 2) == 500000000
    print("PASS")

_check()
```

**Editorial:** Binary search on the answer. Hours needed at speed `k` is `sum(ceil(p / k))`, which only decreases as `k` grows — a monotonic predicate. Search speeds in `[1, max(piles)]`, keeping the smallest `k` whose total hours fit within `h`. Each feasibility check is O(n), for O(n · log(max(piles))) time and O(1) space.
