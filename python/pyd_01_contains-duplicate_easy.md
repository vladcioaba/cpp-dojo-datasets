## challenge: Contains Duplicate
tags: array, hash-table
track: python
lang: python
difficulty: easy

Given a list `nums`, return `True` if any value appears at least twice, and `False` if every element is distinct.

Constraints: `0 <= len(nums) <= 10^5`, values may be negative.

Example: `nums = [1, 2, 3, 1]` → `True` (the `1` repeats). `nums = [1, 2, 3, 4]` → `False`.

hint: Sorting works in O(n log n) — but a set can spot a repeat in a single pass.
hint: A `set` remembers what you've already seen and answers membership in O(1).
hint: Scan left to right; if the current value is already in the set, return early.

```python
# starter
def contains_duplicate(nums):
    ...
```

```python
def contains_duplicate(nums):
    seen = set()
    for x in nums:
        if x in seen:
            return True
        seen.add(x)
    return False
```

```python
# harness
#__USER__
def _check():
    assert contains_duplicate([1, 2, 3, 1]) is True
    assert contains_duplicate([1, 2, 3, 4]) is False
    assert contains_duplicate([1, 1, 1, 3, 3, 4, 3, 2, 4, 2]) is True
    assert contains_duplicate([]) is False
    assert contains_duplicate([7]) is False
    assert contains_duplicate([0, 0]) is True
    print("PASS")

_check()
```

**Editorial:** Walk the list once, keeping a set of values already seen. The moment you meet a value that is already in the set you have a duplicate, so return `True`; otherwise add it and continue. Returning `False` after the loop means all elements were unique. O(n) time, O(n) space — better than the O(n log n) sort-and-compare-neighbors alternative.
