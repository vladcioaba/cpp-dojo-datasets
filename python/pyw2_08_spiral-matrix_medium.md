## challenge: Spiral Matrix
tags: matrix, array
track: python
lang: python
difficulty: medium

Given an `m x n` matrix (a list of lists), return a flat list of all elements visited in clockwise spiral order: across the top row, down the right column, back across the bottom row, up the left column, then repeat one layer in.

Constraints: `0 <= m, n <= 100`; an empty matrix yields `[]`.

Example: `matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]` → `[1, 2, 3, 6, 9, 8, 7, 4, 5]`.

hint: Maintain four boundaries — `top`, `bottom`, `left`, `right` — and shrink one after consuming each edge.
hint: The traversal order is: top row left→right, right column top→bottom, bottom row right→left, left column bottom→top.
hint: For non-square matrices, re-check `top <= bottom` before the bottom row and `left <= right` before the left column, or you'll double-count a lone row or column.

```python
# starter
def spiral_order(matrix):
    ...
```

```python
def spiral_order(matrix):
    if not matrix or not matrix[0]:
        return []
    out = []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1
    while top <= bottom and left <= right:
        for c in range(left, right + 1):
            out.append(matrix[top][c])
        top += 1
        for r in range(top, bottom + 1):
            out.append(matrix[r][right])
        right -= 1
        if top <= bottom:
            for c in range(right, left - 1, -1):
                out.append(matrix[bottom][c])
            bottom -= 1
        if left <= right:
            for r in range(bottom, top - 1, -1):
                out.append(matrix[r][left])
            left += 1
    return out
```

```python
# harness
#__USER__
def _check():
    assert spiral_order([[1, 2, 3], [4, 5, 6], [7, 8, 9]]) == [1, 2, 3, 6, 9, 8, 7, 4, 5]
    assert spiral_order([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]]) == [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]
    assert spiral_order([[1, 2, 3]]) == [1, 2, 3]
    assert spiral_order([[1], [2], [3]]) == [1, 2, 3]
    assert spiral_order([[7]]) == [7]
    assert spiral_order([[1, 2], [3, 4]]) == [1, 2, 4, 3]
    assert spiral_order([]) == []
    print("PASS")

_check()
```

**Editorial:** Peel the matrix layer by layer with four shrinking boundaries. Each pass consumes the top row, right column, bottom row, and left column, tightening the corresponding bound after each edge. The two mid-loop guards prevent re-reading a single remaining row or column in non-square inputs — the classic bug in this problem. O(mn) time, O(1) extra space beyond the output.
