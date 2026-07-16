## challenge: Squares of a Sorted Array
tags: array, two-pointers
track: python
lang: python
difficulty: easy

Given a list `nums` sorted in non-decreasing order (it may contain negatives), return a list of the squares of each number, also sorted in non-decreasing order. Aim for O(n) — squaring then sorting is the O(n log n) baseline.

Constraints: `1 <= len(nums) <= 10^4`, `-10^4 <= nums[i] <= 10^4`, input is sorted.

Example: `nums = [-4, -1, 0, 3, 10]` → `[0, 1, 9, 16, 100]`.

hint: After squaring, the biggest values sit at the two *ends* of the input, not in the middle.
hint: Compare `abs(nums[lo])` with `abs(nums[hi])` — the larger absolute value produces the next-largest square.
hint: Fill the output array from the back while moving `lo` and `hi` inward.

```python
# starter
def sorted_squares(nums):
    ...
```

```python
def sorted_squares(nums):
    n = len(nums)
    out = [0] * n
    lo, hi = 0, n - 1
    for k in range(n - 1, -1, -1):
        if abs(nums[lo]) > abs(nums[hi]):
            out[k] = nums[lo] ** 2
            lo += 1
        else:
            out[k] = nums[hi] ** 2
            hi -= 1
    return out
```

```python
# harness
#__USER__
def _check():
    assert sorted_squares([-4, -1, 0, 3, 10]) == [0, 1, 9, 16, 100]
    assert sorted_squares([-7, -3, 2, 3, 11]) == [4, 9, 9, 49, 121]
    assert sorted_squares([-5, -3, -2]) == [4, 9, 25]
    assert sorted_squares([1, 2, 3]) == [1, 4, 9]
    assert sorted_squares([0]) == [0]
    assert sorted_squares([-1, 1]) == [1, 1]
    print("PASS")

_check()
```

**Editorial:** The squared values form a valley: large at both ends, smallest near zero. Two pointers at the ends pick the larger absolute value each step and write it into the output from the back. One pass, O(n) time, O(n) output space — no sort needed.
