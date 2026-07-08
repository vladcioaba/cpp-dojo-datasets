## challenge: Transpose a Matrix
tags: zip, unpacking
track: python
lang: python
difficulty: easy

Given a rectangular matrix as a list of equal-length rows, return its transpose (rows become columns).

Constraints: all rows have the same length; the matrix may be empty.

Example: `[[1, 2, 3], [4, 5, 6]]` → `[[1, 4], [2, 5], [3, 6]]`.

hint: `zip` pairs up the i-th element of each of its arguments.
hint: Unpack the rows into separate arguments with `zip(*matrix)`.
hint: `zip` yields tuples — wrap each in `list(...)` to get lists back.

```python
# starter
def transpose(matrix):
    ...
```

```python
def transpose(matrix):
    return [list(row) for row in zip(*matrix)]
```

```python
# harness
#__USER__
def _check():
    assert transpose([[1, 2, 3], [4, 5, 6]]) == [[1, 4], [2, 5], [3, 6]]
    assert transpose([[1]]) == [[1]]
    assert transpose([[1, 2], [3, 4]]) == [[1, 3], [2, 4]]
    assert transpose([[1], [2], [3]]) == [[1, 2, 3]]
    assert transpose([]) == []
    print("PASS")

_check()
```

**Editorial:** `zip(*matrix)` unpacks the rows into positional arguments, and `zip` then reads one element from each row per step — precisely a column. It replaces a double `for i in range... for j in range...` index dance with one expression, and correctly produces `[]` for an empty matrix.
