## challenge: Longest Consecutive Sequence
tags: array, hash-table, union-find
track: python
lang: python
difficulty: hard

Given an unsorted list `nums`, return the length of the longest run of consecutive integers (values differing by 1), regardless of their positions in the list. Your algorithm must run in O(n) time.

Constraints: `0 <= len(nums) <= 10^5`, values may repeat and may be negative.

Example: `nums = [100, 4, 200, 1, 3, 2]` → `4` (the run `1, 2, 3, 4`). `nums = [0, 3, 7, 2, 5, 8, 4, 6, 0, 1]` → `9`.

hint: Sorting is O(n log n) — a set gives O(1) membership so you can grow runs directly.
hint: Only start counting a run from a value that has no predecessor present.
hint: For each such start, walk upward `value + 1, value + 2, ...` while the set contains them.

```python
# starter
def longest_consecutive(nums):
    ...
```

```python
def longest_consecutive(nums):
    num_set = set(nums)
    best = 0
    for n in num_set:
        if n - 1 not in num_set:
            length = 1
            while n + length in num_set:
                length += 1
            best = max(best, length)
    return best
```

```python
# harness
#__USER__
def _check():
    assert longest_consecutive([100, 4, 200, 1, 3, 2]) == 4
    assert longest_consecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]) == 9
    assert longest_consecutive([]) == 0
    assert longest_consecutive([1]) == 1
    assert longest_consecutive([1, 2, 0, 1]) == 3
    assert longest_consecutive([9, 1, 4, 7, 3, -1, 0, 5, 8, -1, 6]) == 7
    print("PASS")

_check()
```

**Editorial:** Load all values into a set for O(1) lookups. A number begins a consecutive run only if `n - 1` is absent, so from each such start you walk upward `n + 1, n + 2, ...` counting how far the run extends. Because every value is only ever walked as part of exactly one run, the total work is O(n) despite the nested loop, using O(n) space — beating the O(n log n) sort-based approach.
