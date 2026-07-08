## challenge: Top-K Most Common
tags: counter, collections
track: python
lang: python
difficulty: medium

Return the `k` most frequently occurring elements of `items`, ordered from most to least common. If `k` exceeds the number of distinct elements, return them all.

Constraints: elements are hashable; ties keep first-seen order; `items` may be empty.

Example: `items = [1, 1, 1, 2, 2, 3], k = 2` → `[1, 2]`.

hint: `collections.Counter` tallies frequencies in one pass.
hint: `Counter(items).most_common(k)` returns `(element, count)` pairs already sorted by count.
hint: You only want the elements — unpack and discard the count with `for item, _ in ...`.

```python
# starter
def most_common_k(items, k):
    ...
```

```python
from collections import Counter

def most_common_k(items, k):
    return [item for item, _ in Counter(items).most_common(k)]
```

```python
# harness
#__USER__
def _check():
    assert most_common_k([1, 1, 1, 2, 2, 3], 2) == [1, 2]
    assert most_common_k([], 3) == []
    assert most_common_k([5], 1) == [5]
    assert most_common_k(['a', 'a', 'b', 'b', 'b'], 1) == ['b']
    assert most_common_k([1, 2], 5) == [1, 2]
    print("PASS")

_check()
```

**Editorial:** `Counter` builds the frequency table in a single pass, and `.most_common(k)` returns the top-k `(element, count)` pairs already sorted by descending count (ties broken by insertion order). A comprehension strips off the counts. Doing this by hand means a manual dict tally plus a sort with a custom key — `Counter` packages both.
