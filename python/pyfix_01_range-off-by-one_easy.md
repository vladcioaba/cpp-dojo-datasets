## challenge: fix: Last page never prints
tags: code-review, debugging, off-by-one
track: python
lang: python
difficulty: easy

Code review found a bug: printing pages 1–3 only ever produces pages 1 and 2, and printing a single page (5–5) produces nothing at all. Find and fix it — keep the function signature.

hint: Look at how the sequence of page numbers is generated.
hint: Classic off-by-one — which end of the interval does `range` include?
hint: `range(a, b)` stops at `b - 1`; an inclusive upper bound needs `b + 1`.

```python
# starter
def pages_to_print(first, last):
    """Every page number from `first` to `last`, inclusive."""
    if first > last:
        return []
    return list(range(first, last))
```

```python
def pages_to_print(first, last):
    """Every page number from `first` to `last`, inclusive."""
    if first > last:
        return []
    return list(range(first, last + 1))
```

```python
# harness
#__USER__
def _check():
    assert pages_to_print(1, 3) == [1, 2, 3], "last page missing"
    assert pages_to_print(5, 5) == [5], "single-page job printed nothing"
    assert pages_to_print(4, 2) == []
    assert pages_to_print(0, 1) == [0, 1]
    print("PASS")

_check()
```

**Editorial:** `range(first, last)` is half-open — it stops at `last - 1` — so the final page is always dropped, and a one-page request (`first == last`) yields an empty range. The fix is `range(first, last + 1)`. A reviewer spots this by checking interval endpoints against the docstring ("inclusive") and by mentally running the degenerate case `first == last`, which is where half-open/closed confusion always shows first.
