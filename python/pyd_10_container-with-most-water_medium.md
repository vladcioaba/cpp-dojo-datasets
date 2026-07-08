## challenge: Container With Most Water
tags: array, two-pointers, greedy
track: python
lang: python
difficulty: medium

Given a list `height` where each value is the height of a vertical line at that index, pick two lines that together with the x-axis form a container holding the most water. Return that maximum area. The area between lines `i` and `j` is `min(height[i], height[j]) * (j - i)`.

Constraints: `2 <= len(height) <= 10^5`, `0 <= height[i] <= 10^4`.

Example: `height = [1, 8, 6, 2, 5, 4, 8, 3, 7]` → `49`. `height = [1, 1]` → `1`.

hint: The widest container uses the two ends; narrowing loses width, so it must gain height to help.
hint: Two pointers at the ends, moving inward, examine only promising containers.
hint: Always move the pointer at the shorter line — keeping it can never beat the current area.

```python
# starter
def max_area(height):
    ...
```

```python
def max_area(height):
    i, j = 0, len(height) - 1
    best = 0
    while i < j:
        h = min(height[i], height[j])
        best = max(best, h * (j - i))
        if height[i] < height[j]:
            i += 1
        else:
            j -= 1
    return best
```

```python
# harness
#__USER__
def _check():
    assert max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]) == 49
    assert max_area([1, 1]) == 1
    assert max_area([4, 3, 2, 1, 4]) == 16
    assert max_area([1, 2, 1]) == 2
    assert max_area([2, 3, 4, 5, 18, 17, 6]) == 17
    assert max_area([1, 2, 4, 3]) == 4
    print("PASS")

_check()
```

**Editorial:** Start with the widest possible container (both ends) and shrink inward. The area is bounded by the shorter of the two lines, so moving the taller line inward can only reduce width without lifting that bound — the potential gain lives behind the shorter line. Thus always advance the pointer at the shorter side, tracking the best area seen. O(n) time, O(1) space, versus O(n²) brute force over all pairs.
