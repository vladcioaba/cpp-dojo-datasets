## challenge: Search in Rotated Sorted Array
tags: binary-search, array
track: python
lang: python
difficulty: hard

An ascending sorted array of distinct integers has been rotated at an unknown pivot (for example `[0,1,2,4,5,6,7]` might become `[4,5,6,7,0,1,2]`). Given such an array `nums` and a `target`, return the index of `target`, or `-1` if it is absent. Run in O(log n).

Constraints: `1 <= len(nums) <= 5000`, `-10^4 <= nums[i], target <= 10^4`, all values distinct.

Example: `nums = [4, 5, 6, 7, 0, 1, 2], target = 0` → `4`.

hint: Standard binary search breaks because the array is not fully sorted — but half of it always is.
hint: At each step, one side of `mid` is sorted; check which by comparing `nums[lo]` with `nums[mid]`.
hint: If the target lies within the sorted side's range, search there; otherwise search the other side.

```python
# starter
def search(nums, target):
    ...
```

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

```python
# harness
#__USER__
def _check():
    assert search([4, 5, 6, 7, 0, 1, 2], 0) == 4
    assert search([4, 5, 6, 7, 0, 1, 2], 3) == -1
    assert search([1], 0) == -1
    assert search([1], 1) == 0
    assert search([5, 1, 3], 5) == 0
    assert search([6, 7, 8, 1, 2, 3, 4, 5], 8) == 2
    assert search([3, 4, 5, 6, 7, 8, 9, 1, 2], 2) == 8
    print("PASS")

_check()
```

**Editorial:** Modified binary search. Because the rotation splits the array into two sorted runs, at every `mid` at least one of `[lo, mid]` or `[mid, hi]` is fully sorted — identifiable by comparing `nums[lo]` and `nums[mid]`. If the target falls inside the sorted half's value range, recurse there; otherwise go the other way. O(log n) time, O(1) space.
