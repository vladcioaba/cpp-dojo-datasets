## challenge: Chunk a List
tags: slicing, comprehension
track: python
lang: python
difficulty: easy

Split a list `lst` into consecutive chunks of at most `n` elements each, preserving order. The final chunk may be shorter if the list doesn't divide evenly.

Constraints: `n >= 1`; `lst` may be empty.

Example: `lst = [1, 2, 3, 4, 5], n = 2` → `[[1, 2], [3, 4], [5]]`.

hint: You don't need to iterate element by element — think in slices `lst[i:i+n]`.
hint: `range(start, stop, step)` can stride by `n` at a time.
hint: `[lst[i:i+n] for i in range(0, len(lst), n)]`.

```python
# starter
def chunk(lst, n):
    ...
```

```python
def chunk(lst, n):
    return [lst[i:i+n] for i in range(0, len(lst), n)]
```

```python
# harness
#__USER__
def _check():
    assert chunk([1, 2, 3, 4, 5], 2) == [[1, 2], [3, 4], [5]]
    assert chunk([], 3) == []
    assert chunk([1], 3) == [[1]]
    assert chunk([1, 2, 3, 4], 2) == [[1, 2], [3, 4]]
    assert chunk([1, 2, 3], 1) == [[1], [2], [3]]
    print("PASS")

_check()
```

**Editorial:** Striding `range(0, len(lst), n)` yields every chunk's start index; slicing `lst[i:i+n]` grabs the window and safely clamps at the end, so the short final chunk needs no special case. Slicing beats a manual index-and-append loop: fewer off-by-one opportunities and no mutable accumulator to manage.
