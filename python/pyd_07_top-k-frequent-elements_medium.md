## challenge: Top K Frequent Elements
tags: hash-table, bucket-sort, heap
track: python
lang: python
difficulty: medium

Given a list `nums` and an integer `k`, return the `k` elements that appear most frequently. The answer is guaranteed to be unique; you may return it in any order.

Constraints: `1 <= len(nums) <= 10^5`, `1 <= k <= number of distinct values in nums`.

Example: `nums = [1, 1, 1, 2, 2, 3], k = 2` → `[1, 2]` (order not significant). `nums = [1], k = 1` → `[1]`.

hint: First you need each value's frequency; a counter gives that in one pass.
hint: A frequency can be at most `len(nums)`, so it can index a bucket array directly.
hint: Place each value in a bucket by its count, then read buckets from highest count down, collecting until you have `k`.

```python
# starter
def top_k_frequent(nums, k):
    ...
```

```python
def top_k_frequent(nums, k):
    from collections import Counter
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, freq in count.items():
        buckets[freq].append(num)
    result = []
    for freq in range(len(buckets) - 1, 0, -1):
        for num in buckets[freq]:
            result.append(num)
            if len(result) == k:
                return result
    return result
```

```python
# harness
#__USER__
def _check():
    assert sorted(top_k_frequent([1, 1, 1, 2, 2, 3], 2)) == [1, 2]
    assert sorted(top_k_frequent([1], 1)) == [1]
    assert sorted(top_k_frequent([1, 2], 2)) == [1, 2]
    assert sorted(top_k_frequent([4, 4, 4, 5, 5, 6], 2)) == [4, 5]
    assert sorted(top_k_frequent([-1, -1, -2, -2, -2, 3], 1)) == [-2]
    assert sorted(top_k_frequent([5, 5, 5, 5], 1)) == [5]
    print("PASS")

_check()
```

**Editorial:** Count occurrences with a `Counter`, then bucket-sort by frequency: index `buckets[f]` holds every value that occurs exactly `f` times. Since no frequency exceeds `len(nums)`, walking the buckets from the highest index downward yields values in descending frequency order; collect them until you have `k`. O(n) time and O(n) space — a heap-based approach is O(n log k) instead.
