## challenge: Number of Islands
tags: graph, dfs, bfs, matrix, union-find
track: python
lang: python
difficulty: medium

Given a 2D grid of `'1'` (land) and `'0'` (water) characters, return the number of islands. An island is a maximal group of land cells connected horizontally or vertically. The grid's outside is water.

Constraints: `1 <= rows, cols <= 300`, each cell is `'0'` or `'1'`.

Example: the grid `[['1','1','0'], ['1','0','0'], ['0','0','1']]` → `2`.

hint: Scan every cell; each time you meet an unseen `'1'`, you have found a new island.
hint: From that cell, flood-fill (DFS or BFS) all four-directionally connected land, marking it visited.
hint: Marking visited cells as `'0'` in place avoids needing a separate visited set.

```python
# starter
def num_islands(grid):
    ...
```

```python
def num_islands(grid):
    if not grid or not grid[0]:
        return 0
    rows, cols = len(grid), len(grid[0])
    count = 0
    def sink(r, c):
        if r < 0 or c < 0 or r >= rows or c >= cols or grid[r][c] != '1':
            return
        grid[r][c] = '0'
        sink(r + 1, c)
        sink(r - 1, c)
        sink(r, c + 1)
        sink(r, c - 1)
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                sink(r, c)
    return count
```

```python
# harness
#__USER__
def _check():
    g1 = [
        ['1', '1', '1', '1', '0'],
        ['1', '1', '0', '1', '0'],
        ['1', '1', '0', '0', '0'],
        ['0', '0', '0', '0', '0'],
    ]
    assert num_islands(g1) == 1
    g2 = [
        ['1', '1', '0', '0', '0'],
        ['1', '1', '0', '0', '0'],
        ['0', '0', '1', '0', '0'],
        ['0', '0', '0', '1', '1'],
    ]
    assert num_islands(g2) == 3
    assert num_islands([['0']]) == 0
    assert num_islands([['1']]) == 1
    assert num_islands([['1', '0', '1', '0', '1']]) == 3
    print("PASS")

_check()
```

**Editorial:** Treat land cells as nodes connected to their four neighbours. Walk the grid once; each unvisited `'1'` starts a new island, and a flood fill sinks the whole connected component (rewriting cells to `'0'`) so it is not counted again. O(rows·cols) time; recursion depth is bounded by the number of land cells.
