## challenge: Merge Two Sorted Lists
tags: array, two-pointers
track: python
lang: python
difficulty: easy

Given two lists `a` and `b`, each already sorted in non-decreasing order, merge them into a single sorted list and return it. Do not use `sort()` — produce the result in one linear pass.

Constraints: `0 <= len(a), len(b) <= 10^4`, elements are integers (duplicates allowed).

Example: `a = [1, 2, 4], b = [1, 3, 4]` → `[1, 1, 2, 3, 4, 4]`.

hint: Both inputs are sorted — the smallest remaining element is always at the front of one of them.
hint: Keep an index into each list; compare the two front elements and append the smaller one.
hint: When one index runs off the end, the rest of the other list can be appended wholesale.

```python
# starter
def merge_sorted(a, b):
    ...
```

```python
def merge_sorted(a, b):
    i, j = 0, 0
    out = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            out.append(a[i])
            i += 1
        else:
            out.append(b[j])
            j += 1
    out.extend(a[i:])
    out.extend(b[j:])
    return out
```

```python
# harness
#__USER__
def _check():
    assert merge_sorted([1, 2, 4], [1, 3, 4]) == [1, 1, 2, 3, 4, 4]
    assert merge_sorted([], []) == []
    assert merge_sorted([], [0]) == [0]
    assert merge_sorted([5], []) == [5]
    assert merge_sorted([1, 1, 1], [1, 1]) == [1, 1, 1, 1, 1]
    assert merge_sorted([-3, -1, 2], [-2, 0, 7]) == [-3, -2, -1, 0, 2, 7]
    print("PASS")

_check()
```

**Editorial:** Classic two-pointer merge — the merge step of merge sort. Walk both lists with indices `i` and `j`, always appending the smaller front element, then extend with whichever tail remains. Using `<=` for ties keeps the merge stable. O(n + m) time, O(n + m) space for the output.
