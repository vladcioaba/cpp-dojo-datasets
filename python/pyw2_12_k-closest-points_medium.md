## challenge: K Closest Points to Origin
tags: heap, array
track: python
lang: python
difficulty: medium

Given a list of `points` where `points[i] = [x, y]`, return the `k` points closest to the origin `(0, 0)` by Euclidean distance. The answer may be returned in any order and is guaranteed unique for the given tests.

Constraints: `1 <= k <= len(points) <= 10^4`, `-10^4 <= x, y <= 10^4`.

Example: `points = [[1, 3], [-2, 2]], k = 1` → `[[-2, 2]]` (distance √8 < √10).

hint: Comparing squared distances `x² + y²` avoids the square root entirely — the ordering is identical.
hint: Sorting everything costs O(n log n); a heap of the candidates gets you O(n log k).
hint: `heapq.nsmallest(k, points, key=...)` does the bounded-heap selection in one line.

```python
# starter
def k_closest(points, k):
    ...
```

```python
import heapq

def k_closest(points, k):
    return heapq.nsmallest(k, points, key=lambda p: p[0] * p[0] + p[1] * p[1])
```

```python
# harness
#__USER__
def _check():
    assert sorted(k_closest([[1, 3], [-2, 2]], 1)) == [[-2, 2]]
    assert sorted(k_closest([[3, 3], [5, -1], [-2, 4]], 2)) == [[-2, 4], [3, 3]]
    assert sorted(k_closest([[0, 1], [1, 0]], 2)) == [[0, 1], [1, 0]]
    assert sorted(k_closest([[0, 0]], 1)) == [[0, 0]]
    assert sorted(k_closest([[2, 2], [1, 1], [3, 3]], 2)) == [[1, 1], [2, 2]]
    assert sorted(k_closest([[1, 1], [2, 2], [3, 3], [4, 4], [5, 5]], 3)) == [[1, 1], [2, 2], [3, 3]]
    print("PASS")

_check()
```

**Editorial:** Rank by squared distance — monotone in true distance, so the square root is dead weight. `heapq.nsmallest` maintains a bounded max-heap of size `k` internally: O(n log k) time, O(k) space, better than a full sort when `k << n`. Interviewers may also want the quickselect variant, which reaches O(n) average by partitioning around a pivot distance.
