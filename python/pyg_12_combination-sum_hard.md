## challenge: Combination Sum
tags: backtracking, array, recursion
track: python
lang: python
difficulty: hard

Given a list of distinct positive integers `candidates` and a target integer `target`, return all unique combinations of candidates that sum to `target`. Each candidate may be chosen an unlimited number of times. Two combinations are the same if they use the same multiset of numbers, regardless of order.

Constraints: `1 <= len(candidates) <= 30`, `2 <= candidates[i] <= 40`, `1 <= target <= 40`.

Example: `candidates = [2, 3, 6, 7], target = 7` → `[[2, 2, 3], [7]]`.

hint: Explore choices with backtracking, building one combination on a shared path list.
hint: To avoid permutation duplicates, only pick candidates at or after the current index.
hint: Because reuse is allowed, recurse with the same index; sort candidates so you can prune once one overshoots.

```python
# starter
def combination_sum(candidates, target):
    ...
```

```python
def combination_sum(candidates, target):
    res = []
    candidates = sorted(candidates)
    def bt(start, remain, path):
        if remain == 0:
            res.append(path[:])
            return
        for i in range(start, len(candidates)):
            c = candidates[i]
            if c > remain:
                break
            path.append(c)
            bt(i, remain - c, path)
            path.pop()
    bt(0, target, [])
    return res
```

```python
# harness
#__USER__
def _canon(combos):
    return sorted(sorted(c) for c in combos)

def _check():
    assert _canon(combination_sum([2, 3, 6, 7], 7)) == _canon([[2, 2, 3], [7]])
    assert _canon(combination_sum([2, 3, 5], 8)) == _canon([[2, 2, 2, 2], [2, 3, 3], [3, 5]])
    assert _canon(combination_sum([2], 1)) == _canon([])
    assert _canon(combination_sum([2], 4)) == _canon([[2, 2]])
    assert _canon(combination_sum([3, 5, 8], 11)) == _canon([[3, 8], [3, 3, 5]])
    print("PASS")

_check()
```

**Editorial:** Depth-first backtracking. Passing a `start` index forbids revisiting earlier candidates, which collapses order-only duplicates; recursing with the same index `i` lets a candidate repeat. Sorting enables an early `break` once a candidate exceeds the remaining target. The checker canonicalises by sorting each combination and the list of combinations, so any output order validates.
