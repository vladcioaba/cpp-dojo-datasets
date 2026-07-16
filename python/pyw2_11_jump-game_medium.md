## challenge: Jump Game
tags: greedy, array, dp
track: python
lang: python
difficulty: medium

You are given a list `nums` where `nums[i]` is the maximum number of positions you may jump forward from index `i`. Starting at index `0`, return `True` if you can reach the last index, and `False` otherwise.

Constraints: `1 <= len(nums) <= 10^4`, `0 <= nums[i] <= 10^5`.

Example: `nums = [2, 3, 1, 1, 4]` → `True` (jump 0→1, then 1→4).

hint: You don't need to know *which* jumps to take — only how far right you could possibly be.
hint: Track `reach`, the furthest index attainable so far; from index `i` you can extend it to `i + nums[i]`.
hint: If you ever stand at an index beyond `reach`, you're stranded — that's the only way to fail.

```python
# starter
def can_jump(nums):
    ...
```

```python
def can_jump(nums):
    reach = 0
    for i, step in enumerate(nums):
        if i > reach:
            return False
        reach = max(reach, i + step)
    return True
```

```python
# harness
#__USER__
def _check():
    assert can_jump([2, 3, 1, 1, 4]) is True
    assert can_jump([3, 2, 1, 0, 4]) is False
    assert can_jump([0]) is True
    assert can_jump([0, 1]) is False
    assert can_jump([2, 0, 0]) is True
    assert can_jump([1, 1, 1, 1]) is True
    assert can_jump([5, 0, 0, 0]) is True
    print("PASS")

_check()
```

**Editorial:** Greedy beats DP here. Sweep left to right maintaining `reach`, the furthest reachable index; a position `i > reach` is unreachable, so the answer is `False`, otherwise fold `i + nums[i]` into `reach`. If the scan completes, the last index was covered. The greedy is safe because reachability is monotone — reaching further never hurts. O(n) time, O(1) space, versus O(n²) for naive DP.
