## challenge: Two Sum
tags: array, hash-table
track: python
lang: python
difficulty: easy

Given a list `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one solution exists; you may not use the same element twice.

Constraints: `2 <= len(nums) <= 10^4`, each input has exactly one answer.

Example: `nums = [2, 7, 11, 15], target = 9` → `[0, 1]` (because `2 + 7 == 9`).

hint: Brute force is O(n²) — can a dict remember the numbers you've already seen?
hint: For each `x`, you need `target - x`. Look it up in O(1).
hint: Store `value -> index` as you scan; check for the complement before inserting.

```python
# starter
def two_sum(nums, target):
    ...
```

```python
def two_sum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
    return []
```

```python
# harness
#__USER__
def _check():
    assert sorted(two_sum([2, 7, 11, 15], 9)) == [0, 1]
    assert sorted(two_sum([3, 2, 4], 6)) == [1, 2]
    assert sorted(two_sum([3, 3], 6)) == [0, 1]
    assert sorted(two_sum([-1, -2, -3, -4, -5], -8)) == [2, 4]
    print("PASS")

_check()
```

**Editorial:** One pass with a dict mapping each value to its index. For each element check whether its complement `target - x` was already seen; if so you have the pair. O(n) time, O(n) space — versus the O(n²) nested-loop scan.
