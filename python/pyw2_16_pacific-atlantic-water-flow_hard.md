## challenge: Pacific Atlantic Water Flow
tags: graph, bfs, matrix
track: python
lang: python
difficulty: hard

You are given an `m x n` grid `heights` of land elevations. The Pacific Ocean touches the top and left edges; the Atlantic touches the bottom and right edges. Rain water flows from a cell to a 4-directional neighbor of **equal or lower** height, and into an ocean off the matching edges. Return a list of `[r, c]` coordinates from which water can reach *both* oceans, in any order.

Constraints: `1 <= m, n <= 200`, `0 <= heights[r][c] <= 10^5`.

Example: `heights = [[1, 2], [4, 3]]` → `[[0, 1], [1, 0], [1, 1]]` (only `[0, 0]` is landlocked from the Atlantic).

hint: Tracing water *downhill* from every cell is O((mn)²). Flip it: flood *uphill* from each ocean instead.
hint: Start a search from every Pacific-edge cell and every Atlantic-edge cell, moving only to neighbors with height `>=` the current cell.
hint: Two reachability sets — one per ocean — and the answer is their intersection.

```python
# starter
def pacific_atlantic(heights):
    ...
```

```python
def pacific_atlantic(heights):
    rows, cols = len(heights), len(heights[0])

    def flood(starts):
        seen = set(starts)
        stack = list(starts)
        while stack:
            r, c = stack.pop()
            for nr, nc in ((r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)):
                if (0 <= nr < rows and 0 <= nc < cols
                        and (nr, nc) not in seen
                        and heights[nr][nc] >= heights[r][c]):
                    seen.add((nr, nc))
                    stack.append((nr, nc))
        return seen

    pacific = flood([(r, 0) for r in range(rows)] + [(0, c) for c in range(cols)])
    atlantic = flood([(r, cols - 1) for r in range(rows)] + [(rows - 1, c) for c in range(cols)])
    return [[r, c] for r, c in pacific & atlantic]
```

```python
# harness
#__USER__
def _check():
    grid = [[1, 2, 2, 3, 5], [3, 2, 3, 4, 4], [2, 4, 5, 3, 1], [6, 7, 1, 4, 5], [5, 1, 1, 2, 4]]
    assert sorted(pacific_atlantic(grid)) == [[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]]
    assert sorted(pacific_atlantic([[1]])) == [[0, 0]]
    assert sorted(pacific_atlantic([[1, 2], [4, 3]])) == [[0, 1], [1, 0], [1, 1]]
    assert sorted(pacific_atlantic([[1, 2, 3]])) == [[0, 0], [0, 1], [0, 2]]
    assert sorted(pacific_atlantic([[2, 1], [1, 2]])) == [[0, 0], [0, 1], [1, 0], [1, 1]]
    assert sorted(pacific_atlantic([[10, 10], [10, 10]])) == [[0, 0], [0, 1], [1, 0], [1, 1]]
    print("PASS")

_check()
```

**Editorial:** The key inversion: instead of asking "where can each cell drain to?", flood *from* each ocean, moving uphill (`neighbor >= current`) — a cell an ocean can climb to is exactly a cell that drains to that ocean. Two multi-source searches (BFS or DFS, here an iterative stack) seeded from the ocean edges give two reachability sets; the answer is their intersection. Each search visits every cell once: O(mn) time and space, versus O((mn)²) for per-cell downhill simulation.
