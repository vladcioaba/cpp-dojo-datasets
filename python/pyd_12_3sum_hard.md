## challenge: 3Sum
tags: array, two-pointers, sorting
track: python
lang: python
difficulty: hard

Given a list `nums`, return all unique triplets `[a, b, c]` drawn from distinct indices such that `a + b + c == 0`. The result must not contain duplicate triplets. Order of the triplets and of values within a triplet does not matter.

Constraints: `0 <= len(nums) <= 3000`, `-10^5 <= nums[i] <= 10^5`.

Example: `nums = [-1, 0, 1, 2, -1, -4]` → `[[-1, -1, 2], [-1, 0, 1]]` (order not significant). `nums = [0, 1, 1]` → `[]`.

hint: Sorting first lets you skip duplicates and use a directional two-pointer scan.
hint: Fix one element, then two-sum the remaining suffix toward its negation.
hint: After recording a triplet, advance past equal values on both sides to avoid duplicates.

```python
# starter
def three_sum(nums):
    ...
```

```python
def three_sum(nums):
    nums.sort()
    res = []
    n = len(nums)
    for i in range(n - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        if nums[i] > 0:
            break
        lo, hi = i + 1, n - 1
        while lo < hi:
            s = nums[i] + nums[lo] + nums[hi]
            if s < 0:
                lo += 1
            elif s > 0:
                hi -= 1
            else:
                res.append([nums[i], nums[lo], nums[hi]])
                lo += 1
                hi -= 1
                while lo < hi and nums[lo] == nums[lo - 1]:
                    lo += 1
                while lo < hi and nums[hi] == nums[hi + 1]:
                    hi -= 1
    return res
```

```python
# harness
#__USER__
def _canon(triplets):
    return sorted(sorted(t) for t in triplets)

def _check():
    assert _canon(three_sum([-1, 0, 1, 2, -1, -4])) == _canon([[-1, -1, 2], [-1, 0, 1]])
    assert _canon(three_sum([0, 1, 1])) == []
    assert _canon(three_sum([0, 0, 0])) == [[0, 0, 0]]
    assert _canon(three_sum([])) == []
    assert _canon(three_sum([-2, 0, 1, 1, 2])) == _canon([[-2, 0, 2], [-2, 1, 1]])
    assert _canon(three_sum([-4, -2, -2, -2, 0, 1, 2, 2, 2, 3, 3, 4, 4, 6, 6])) == \
        _canon([[-4, -2, 6], [-4, 0, 4], [-4, 1, 3], [-4, 2, 2], [-2, -2, 4], [-2, 0, 2]])
    print("PASS")

_check()
```

**Editorial:** Sort the array, then fix each value `nums[i]` and search its suffix with two pointers for a pair summing to `-nums[i]`. Moving the left pointer up increases the sum, the right pointer down decreases it, so you converge in linear time per fix. Skip equal values at the fixed index and after each hit to suppress duplicate triplets; once `nums[i] > 0` no triple can sum to zero, so stop early. O(n²) time and O(1) extra space beyond the output.
