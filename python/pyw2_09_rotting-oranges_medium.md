## challenge: Rotting Oranges
tags: bfs, matrix, graph
track: python
lang: python
difficulty: medium

You are given an `m x n` grid where each cell is `0` (empty), `1` (fresh orange), or `2` (rotten orange). Every minute, each fresh orange 4-directionally adjacent to a rotten one becomes rotten. Return the number of minutes until no fresh orange remains, or `-1` if some fresh orange can never rot. Do not mutate the input grid.

Constraints: `1 <= m, n <= 100`, cells are `0`, `1`, or `2`.

Example: `grid = [[2, 1, 1], [1, 1, 0], [0, 1, 1]]` → `4`.

hint: "Every minute, all adjacent at once" is the signature of multi-source BFS — seed the queue with *every* rotten orange.
hint: Process the queue level by level; each level is one minute.
hint: Count fresh oranges up front. If the BFS finishes with fresh ones left, they're unreachable — return -1.

```python
# starter
def oranges_rotting(grid):
    ...
```

```python
from collections import deque

def oranges_rotting(grid):
    rows, cols = len(grid), len(grid[0])
    grid = [row[:] for row in grid]
    q = deque()
    fresh = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                q.append((r, c))
            elif grid[r][c] == 1:
                fresh += 1
    minutes = 0
    while q and fresh:
        for _ in range(len(q)):
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    q.append((nr, nc))
        minutes += 1
    return -1 if fresh else minutes
```

```python
# harness
#__USER__
def _check():
    assert oranges_rotting([[2, 1, 1], [1, 1, 0], [0, 1, 1]]) == 4
    assert oranges_rotting([[2, 1, 1], [0, 1, 1], [1, 0, 1]]) == -1
    assert oranges_rotting([[0, 2]]) == 0
    assert oranges_rotting([[0]]) == 0
    assert oranges_rotting([[1]]) == -1
    assert oranges_rotting([[2, 1, 1, 1]]) == 3
    assert oranges_rotting([[2], [1], [1], [2]]) == 1
    print("PASS")

_check()
```

**Editorial:** Multi-source BFS: enqueue every initially rotten orange, count the fresh ones, then expand in levels — one level per minute. Looping `for _ in range(len(q))` freezes the current frontier so newly rotted oranges wait for the next minute. Guarding the loop with `and fresh` stops the clock the moment the last orange rots, and any leftover `fresh` means an isolated orange → -1. O(mn) time and space.
