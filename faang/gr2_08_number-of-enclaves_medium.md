## challenge: Number of Enclaves
tags: graph, dfs, bfs, matrix
track: faang
difficulty: medium

You are given an `m x n` binary matrix `grid`, where `0` represents sea and `1` represents land. A move consists of walking from one land cell to another adjacent (up, down, left, or right) land cell or walking off the boundary of the grid. Return the number of land cells from which it is impossible to walk off the boundary of the grid in any number of moves.

Constraints: `m == grid.length`, `n == grid[i].length`, `1 <= m, n <= 500`, `grid[i][j]` is `0` or `1`.

Example: `grid = [[0,0,0,0],[1,0,1,0],[0,1,1,0],[0,0,0,0]]` → `3`. Example: `grid = [[0,1,1,0],[0,0,1,0],[0,0,1,0],[0,0,0,0]]` → `0` (all land connects to the boundary).

hint: A land cell can escape iff it is connected to a land cell on the border. Flood-fill inward from all border land cells.
hint: Sink (turn to sea) every land cell reachable from the boundary, then count the land cells that remain.

```cpp
// starter
#include <vector>
int numEnclaves(std::vector<std::vector<int>>& grid);
```

```cpp
#include <functional>
int numEnclaves(std::vector<std::vector<int>>& grid) {
    int m = (int)grid.size(), n = (int)grid[0].size();
    std::function<void(int, int)> dfs = [&](int r, int c) {
        if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != 1) return;
        grid[r][c] = 0;
        dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1);
    };
    for (int i = 0; i < m; ++i) { dfs(i, 0); dfs(i, n - 1); }
    for (int j = 0; j < n; ++j) { dfs(0, j); dfs(m - 1, j); }
    int count = 0;
    for (int i = 0; i < m; ++i)
        for (int j = 0; j < n; ++j)
            count += grid[i][j];
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{{0,0,0,0},{1,0,1,0},{0,1,1,0},{0,0,0,0}}; if (numEnclaves(g) != 3) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{0,1,1,0},{0,0,1,0},{0,0,1,0},{0,0,0,0}}; if (numEnclaves(g) != 0) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{1}}; if (numEnclaves(g) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{0,0,0},{0,1,0},{0,0,0}}; if (numEnclaves(g) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{0,0,0,0,0},{0,1,1,1,0},{0,1,0,1,0},{0,1,1,1,0},{0,0,0,0,0}}; if (numEnclaves(g) != 8) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A land cell can walk off the grid exactly when it is connected (through land) to a border cell. Flood-fill from every land cell on the four borders, sinking each reachable land cell to sea. Any land cell left after this cannot reach the boundary, so the answer is the count of remaining `1`s. Each cell is visited a constant number of times, giving O(m * n) time and O(m * n) recursion space in the worst case.
