## challenge: Partition Equal Subset Sum
tags: dp, array
track: python
lang: python
difficulty: hard

Given a list `nums` of positive integers, return `True` if the list can be split into two subsets with equal sums, and `False` otherwise. Every element must land in exactly one subset.

Constraints: `1 <= len(nums) <= 200`, `1 <= nums[i] <= 100`.

Example: `nums = [1, 5, 11, 5]` → `True` (`[1, 5, 5]` and `[11]` both sum to 11).

hint: If the total sum is odd, no split can exist. If it's even, you're hunting a subset summing to exactly `total // 2` — 0/1 knapsack.
hint: Track the set of subset sums reachable so far; each number extends every previously reachable sum.
hint: Prune sums above the target, and return early the moment the target becomes reachable.

```python
# starter
def can_partition(nums):
    ...
```

```python
def can_partition(nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    reachable = {0}
    for x in nums:
        reachable |= {r + x for r in reachable if r + x <= target}
        if target in reachable:
            return True
    return False
```

```python
# harness
#__USER__
def _check():
    assert can_partition([1, 5, 11, 5]) is True
    assert can_partition([1, 2, 3, 5]) is False
    assert can_partition([1, 1]) is True
    assert can_partition([1]) is False
    assert can_partition([2, 2, 3, 5]) is False
    assert can_partition([3, 3, 3, 4, 5]) is True
    assert can_partition([100]) is False
    print("PASS")

_check()
```

**Editorial:** Reduce to subset-sum: an equal split exists iff some subset hits `total // 2` (impossible when the total is odd). The DP state is simply "which sums are reachable" — a set (or boolean array/bitmask) that each number extends by shifting. Building the extension as a separate set before the union is the 0/1 discipline: it stops one element from being used twice in the same round. O(n · target) time, O(target) space; the bitmask trick (`bits |= bits << x`) makes it startlingly fast in practice.
