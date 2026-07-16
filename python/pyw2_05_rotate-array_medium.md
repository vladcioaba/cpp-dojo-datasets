## challenge: Rotate Array
tags: array, two-pointers
track: python
lang: python
difficulty: medium

Given a list `nums` and a non-negative integer `k`, return a new list equal to `nums` rotated to the right by `k` steps (the last `k` elements wrap around to the front). `k` may be larger than the length of the list.

Constraints: `1 <= len(nums) <= 10^5`, `0 <= k <= 10^9`.

Example: `nums = [1, 2, 3, 4, 5, 6, 7], k = 3` → `[5, 6, 7, 1, 2, 3, 4]`.

hint: Rotating by `len(nums)` steps lands you back where you started — what does that say about huge `k`?
hint: Reduce with `k % n` first, then the answer is just two slices glued together.
hint: The last `k` elements are `nums[-k:]`; everything before them is `nums[:-k]`. Watch the `k == 0` case — `nums[-0:]` is the whole list!

```python
# starter
def rotate_array(nums, k):
    ...
```

```python
def rotate_array(nums, k):
    n = len(nums)
    k %= n
    if k == 0:
        return nums[:]
    return nums[-k:] + nums[:-k]
```

```python
# harness
#__USER__
def _check():
    assert rotate_array([1, 2, 3, 4, 5, 6, 7], 3) == [5, 6, 7, 1, 2, 3, 4]
    assert rotate_array([-1, -100, 3, 99], 2) == [3, 99, -1, -100]
    assert rotate_array([1, 2, 3], 0) == [1, 2, 3]
    assert rotate_array([1, 2, 3], 3) == [1, 2, 3]
    assert rotate_array([1, 2, 3], 10) == [3, 1, 2]
    assert rotate_array([1], 5) == [1]
    print("PASS")

_check()
```

**Editorial:** Normalize `k %= n` since a full-length rotation is the identity, then concatenate the two slices `nums[-k:] + nums[:-k]`. The classic trap is `k == 0`: `nums[-0:]` slices the entire list, so it needs a guard. The in-place alternative (reverse all, reverse first `k`, reverse rest) achieves O(1) extra space; the slice version is O(n) time and space.
