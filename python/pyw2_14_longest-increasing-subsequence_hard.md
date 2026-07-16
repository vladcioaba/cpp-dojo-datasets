## challenge: Longest Increasing Subsequence
tags: dp, binary-search, array
track: python
lang: python
difficulty: hard

Given a list `nums`, return the length of the longest strictly increasing subsequence. A subsequence keeps relative order but need not be contiguous. The O(n²) DP is easy — aim for O(n log n).

Constraints: `0 <= len(nums) <= 2500`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [10, 9, 2, 5, 3, 7, 101, 18]` → `4` (one LIS is `[2, 3, 7, 101]`).

hint: Keep `tails`, where `tails[i]` is the smallest possible tail of an increasing subsequence of length `i + 1`.
hint: `tails` is always sorted — so each new number can be placed with binary search (`bisect_left`).
hint: If the number extends beyond every tail, append it (length grows); otherwise it *replaces* the first tail `>=` it, keeping future options open.

```python
# starter
def length_of_lis(nums):
    ...
```

```python
from bisect import bisect_left

def length_of_lis(nums):
    tails = []
    for x in nums:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)
```

```python
# harness
#__USER__
def _check():
    assert length_of_lis([10, 9, 2, 5, 3, 7, 101, 18]) == 4
    assert length_of_lis([0, 1, 0, 3, 2, 3]) == 4
    assert length_of_lis([7, 7, 7, 7]) == 1
    assert length_of_lis([1]) == 1
    assert length_of_lis([]) == 0
    assert length_of_lis([4, 10, 4, 3, 8, 9]) == 3
    assert length_of_lis([1, 3, 6, 7, 9, 4, 10, 5, 6]) == 6
    print("PASS")

_check()
```

**Editorial:** Patience sorting. `tails[i]` holds the smallest tail of any increasing subsequence of length `i + 1`; that invariant keeps `tails` sorted, so `bisect_left` finds each number's spot in O(log n). Appending means a longer LIS exists; replacing lowers a tail so later numbers have an easier bar to clear. Note `tails` is *not* an actual LIS — only its length is meaningful. `bisect_left` (not `bisect_right`) enforces strict increase by rejecting equal elements. O(n log n) time, O(n) space.
