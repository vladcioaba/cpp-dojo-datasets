## challenge: Non-overlapping Intervals
tags: intervals, greedy
track: python
lang: python
difficulty: hard

Given a list of intervals `[start, end]`, return the minimum number of intervals you must remove so that the remaining intervals are mutually non-overlapping. Intervals that only touch at a point (like `[1, 2]` and `[2, 3]`) do not overlap.

Constraints: `0 <= len(intervals) <= 10^5`, `start < end` for every interval.

Example: `intervals = [[1, 2], [2, 3], [3, 4], [1, 3]]` → `1` (remove `[1, 3]`).

hint: Minimizing removals is the same as maximizing the number of intervals you keep — a scheduling problem in disguise.
hint: Greedy by earliest *end* time: the interval that finishes first leaves the most room for the rest.
hint: Sort by end; keep an interval if its start is at or after the end of the last kept one, otherwise count it as removed.

```python
# starter
def erase_overlap_intervals(intervals):
    ...
```

```python
def erase_overlap_intervals(intervals):
    removed = 0
    prev_end = float("-inf")
    for start, end in sorted(intervals, key=lambda iv: iv[1]):
        if start >= prev_end:
            prev_end = end
        else:
            removed += 1
    return removed
```

```python
# harness
#__USER__
def _check():
    assert erase_overlap_intervals([[1, 2], [2, 3], [3, 4], [1, 3]]) == 1
    assert erase_overlap_intervals([[1, 2], [1, 2], [1, 2]]) == 2
    assert erase_overlap_intervals([[1, 2], [2, 3]]) == 0
    assert erase_overlap_intervals([]) == 0
    assert erase_overlap_intervals([[1, 2]]) == 0
    assert erase_overlap_intervals([[1, 100], [11, 22], [1, 11], [2, 12]]) == 2
    print("PASS")

_check()
```

**Editorial:** This is interval scheduling maximization flipped around: `removals = n - (max compatible set)`. The greedy that sorts by end time and always keeps the earliest-finishing compatible interval is provably optimal — an exchange argument shows any optimal solution can be rewritten to use the earliest end without losing intervals. Sorting by *start* instead is the classic wrong move. O(n log n) time, O(1) extra space.
