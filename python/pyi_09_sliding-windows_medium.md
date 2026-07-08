## challenge: Sliding Windows
tags: zip, unpacking
track: python
lang: python
difficulty: medium

Return all consecutive windows of length `n` over `lst`, as a list of tuples. Windows overlap and slide one position at a time. If `lst` is shorter than `n`, there are no windows.

Constraints: `n >= 1`; `lst` may be empty.

Example: `lst = [1, 2, 3, 4], n = 2` → `[(1, 2), (2, 3), (3, 4)]`.

hint: Window `i` is `lst[i], lst[i+1], ..., lst[i+n-1]` — the same list shifted by 0, 1, ..., n-1.
hint: `zip` stops at its shortest argument, which trims the ragged tail automatically.
hint: `zip(*(lst[i:] for i in range(n)))` zips `n` progressively-shifted slices.

```python
# starter
def windows(lst, n):
    ...
```

```python
def windows(lst, n):
    return list(zip(*(lst[i:] for i in range(n))))
```

```python
# harness
#__USER__
def _check():
    assert windows([1, 2, 3, 4], 2) == [(1, 2), (2, 3), (3, 4)]
    assert windows([1, 2, 3, 4], 3) == [(1, 2, 3), (2, 3, 4)]
    assert windows([], 2) == []
    assert windows([1], 2) == []
    assert windows([1, 2, 3], 1) == [(1,), (2,), (3,)]
    assert windows([1, 2], 2) == [(1, 2)]
    print("PASS")

_check()
```

**Editorial:** Shifting the list by `0..n-1` and zipping the shifted copies lines up each window as one tuple; because `zip` halts at the shortest slice, the ragged tail is dropped and a too-short list yields no windows — no bounds arithmetic. This is the classic `zip`-of-offset-slices windowing trick.
