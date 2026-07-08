## challenge: Sort by Computed Key
tags: sorting-key, lambda
track: python
lang: python
difficulty: medium

Sort a list of 2D points (given as `(x, y)` tuples) by their squared distance from the origin, ascending. Points that are equally distant keep their original relative order.

Constraints: coordinates are integers (may be negative); the list may be empty.

Example: `[(3, 4), (1, 1), (0, 2)]` → `[(1, 1), (0, 2), (3, 4)]` (distances² are 2, 4, 25).

hint: Don't reorder by hand — pass `sorted` a `key` function that maps each point to its sort value.
hint: Squared distance `x*x + y*y` avoids a `sqrt` and orders identically.
hint: `sorted(points, key=lambda p: p[0]**2 + p[1]**2)`.

```python
# starter
def sort_by_distance(points):
    ...
```

```python
def sort_by_distance(points):
    return sorted(points, key=lambda p: p[0]**2 + p[1]**2)
```

```python
# harness
#__USER__
def _check():
    assert sort_by_distance([(3, 4), (1, 1), (0, 2)]) == [(1, 1), (0, 2), (3, 4)]
    assert sort_by_distance([]) == []
    assert sort_by_distance([(5, 5)]) == [(5, 5)]
    assert sort_by_distance([(1, 0), (0, 1)]) == [(1, 0), (0, 1)]
    assert sort_by_distance([(-3, -4), (1, 1)]) == [(1, 1), (-3, -4)]
    print("PASS")

_check()
```

**Editorial:** A `key` function is computed once per element and cached, so `sorted` orders by squared distance without you touching comparisons. Squaring instead of `sqrt` keeps the ordering and stays in integer math. Because Python's sort is stable, equidistant points keep input order for free — far cleaner than a comparator that recomputes distance on every compare.
