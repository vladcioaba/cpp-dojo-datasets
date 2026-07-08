## challenge: Running Sum
tags: itertools, accumulate
track: python
lang: python
difficulty: medium

Return the running (cumulative) sum of `nums`: element `i` of the result is the sum of `nums[0..i]` inclusive.

Constraints: numbers may be negative; `nums` may be empty.

Example: `[1, 2, 3, 4]` → `[1, 3, 6, 10]`.

hint: Each output equals the previous output plus the current input — a running fold.
hint: `itertools.accumulate` does exactly this fold, defaulting to addition.
hint: It returns an iterator — wrap it in `list(...)`.

```python
# starter
def running_sum(nums):
    ...
```

```python
from itertools import accumulate

def running_sum(nums):
    return list(accumulate(nums))
```

```python
# harness
#__USER__
def _check():
    assert running_sum([1, 2, 3, 4]) == [1, 3, 6, 10]
    assert running_sum([]) == []
    assert running_sum([5]) == [5]
    assert running_sum([1, -1, 1, -1]) == [1, 0, 1, 0]
    assert running_sum([0, 0, 0]) == [0, 0, 0]
    print("PASS")

_check()
```

**Editorial:** `itertools.accumulate` yields the partial reductions of a sequence, defaulting to `operator.add`, so the cumulative sum is a direct call. It streams lazily (one addition per step, O(n)) and handles empty input cleanly, versus maintaining a `total` variable and appending inside a loop.
