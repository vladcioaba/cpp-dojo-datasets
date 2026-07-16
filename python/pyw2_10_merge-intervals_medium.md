## challenge: Merge Intervals
tags: intervals, array
track: python
lang: python
difficulty: medium

Given a list of intervals `[start, end]` in arbitrary order, merge all overlapping intervals and return the merged list sorted by start. Intervals that merely touch (one ends exactly where the next begins) count as overlapping.

Constraints: `0 <= len(intervals) <= 10^4`, `start <= end` for every interval.

Example: `intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]` → `[[1, 6], [8, 10], [15, 18]]`.

hint: Sort by start first — then any overlap must be with the interval you just emitted.
hint: Compare each interval's start with the end of the last merged interval.
hint: On overlap, don't just replace the end — take the max, or a nested interval like `[2, 3]` inside `[1, 10]` will shrink it.

```python
# starter
def merge_intervals(intervals):
    ...
```

```python
def merge_intervals(intervals):
    merged = []
    for start, end in sorted(intervals):
        if merged and start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```

```python
# harness
#__USER__
def _check():
    assert merge_intervals([[1, 3], [2, 6], [8, 10], [15, 18]]) == [[1, 6], [8, 10], [15, 18]]
    assert merge_intervals([[1, 4], [4, 5]]) == [[1, 5]]
    assert merge_intervals([[5, 6], [1, 3], [2, 4]]) == [[1, 4], [5, 6]]
    assert merge_intervals([[1, 4]]) == [[1, 4]]
    assert merge_intervals([[1, 10], [2, 3], [4, 5]]) == [[1, 10]]
    assert merge_intervals([]) == []
    assert merge_intervals([[-5, -2], [-3, 0], [1, 2]]) == [[-5, 0], [1, 2]]
    print("PASS")

_check()
```

**Editorial:** Sort by start, then sweep once: if the current interval starts at or before the end of the last merged one, extend that end with `max` (the max matters for nested intervals); otherwise start a new merged interval. Sorting dominates at O(n log n); the sweep is O(n). This sort-then-sweep pattern is the backbone of nearly every interval problem.
