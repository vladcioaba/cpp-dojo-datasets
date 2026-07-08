## challenge: House Robber
tags: dynamic-programming, array
track: python
lang: python
difficulty: medium

Given a list `nums` of non-negative integers representing the money in each house along a street, return the maximum amount you can rob without ever robbing two adjacent houses.

Constraints: `0 <= len(nums) <= 100`, `0 <= nums[i] <= 400`.

Example: `nums = [2, 7, 9, 3, 1]` → `12` (rob houses `2 + 9 + 1`).

hint: At each house you either skip it or rob it and skip its neighbour.
hint: `best(i) = max(best(i-1), best(i-2) + nums[i])`.
hint: Only the previous two answers matter, so track two rolling values.

```python
# starter
def rob(nums):
    ...
```

```python
def rob(nums):
    prev, cur = 0, 0
    for x in nums:
        prev, cur = cur, max(cur, prev + x)
    return cur
```

```python
# harness
#__USER__
def _check():
    assert rob([1, 2, 3, 1]) == 4
    assert rob([2, 7, 9, 3, 1]) == 12
    assert rob([]) == 0
    assert rob([5]) == 5
    assert rob([2, 1, 1, 2]) == 4
    assert rob([200, 3, 140, 20, 10]) == 350
    print("PASS")

_check()
```

**Editorial:** Classic linear DP. Let `cur` be the best take through the current house and `prev` the best through the house before it. Robbing the current house adds `nums[i]` to `prev` (its non-adjacent predecessor); skipping keeps `cur`. Two rolling variables give O(n) time and O(1) space.
