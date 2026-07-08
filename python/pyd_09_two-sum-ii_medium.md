## challenge: Two Sum II - Input Array Is Sorted
tags: array, two-pointers, binary-search
track: python
lang: python
difficulty: medium

Given a list `numbers` sorted in non-decreasing order and an integer `target`, find the two elements that add up to `target` and return their 1-based indices `[i, j]` with `i < j`. Exactly one solution exists, and you must use O(1) extra space.

Constraints: `2 <= len(numbers) <= 3 * 10^4`, `numbers` is sorted ascending, exactly one answer.

Example: `numbers = [2, 7, 11, 15], target = 9` → `[1, 2]`. `numbers = [2, 3, 4], target = 6` → `[1, 3]`.

hint: The array is sorted — exploit that instead of hashing.
hint: One pointer at the start, one at the end; their sum tells you which way to move.
hint: If the pair's sum is too small advance the left pointer; if too big retreat the right one.

```python
# starter
def two_sum_sorted(numbers, target):
    ...
```

```python
def two_sum_sorted(numbers, target):
    i, j = 0, len(numbers) - 1
    while i < j:
        s = numbers[i] + numbers[j]
        if s == target:
            return [i + 1, j + 1]
        elif s < target:
            i += 1
        else:
            j -= 1
    return []
```

```python
# harness
#__USER__
def _check():
    assert two_sum_sorted([2, 7, 11, 15], 9) == [1, 2]
    assert two_sum_sorted([2, 3, 4], 6) == [1, 3]
    assert two_sum_sorted([-1, 0], -1) == [1, 2]
    assert two_sum_sorted([1, 2, 3, 4, 4, 9, 56, 90], 8) == [4, 5]
    assert two_sum_sorted([5, 25, 75], 100) == [2, 3]
    assert two_sum_sorted([1, 2, 3, 4], 100) == []
    print("PASS")

_check()
```

**Editorial:** Because the array is sorted, put one pointer at each end. Their sum is monotonic in the pointers: if it is below the target the only way to increase it is to move the left pointer right; if above, move the right pointer left. When the sum matches, return the 1-based indices. O(n) time and O(1) space, beating a hash map's O(n) memory.
