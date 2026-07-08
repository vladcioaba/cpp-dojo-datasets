## challenge: Max Area of Island

tags: graph, dfs, matrix, flood-fill
track: faang
difficulty: easy

You are given an `m x n` binary grid where `1` is land and `0` is water. An island is a maximal group of `1`s connected 4-directionally. The area of an island is the number of cells it contains. Return the area of the largest island, or `0` if there is none. You may mutate the grid.

Constraints: `1 <= m, n <= 50`, each `grid[r][c]` is `0` or `1`.

Example: `grid = [[1,1,0,0],[1,0,0,1],[0,0,1,1]]` → `3` (the bottom-right island has three cells). Example: a grid of all `0`s → `0`.

hint: The largest island is the biggest connected component of land — compute each component's size and keep the maximum.
hint: Flood fill from every unvisited land cell, sinking cells to water as you count them so no cell is tallied twice.
hint: Have the recursion return the size it discovered: `1 + sum of the four neighbour calls`.

```cpp
// starter
#include <vector>
int maxAreaOfIsland(std::vector<std::vector<int>>& grid);
```

```cpp
int maxAreaOfIsland(std::vector<std::vector<int>>& grid) {
    int m = (int)grid.size();
    if (m == 0) return 0;
    int n = (int)grid[0].size();
    std::function<int(int, int)> dfs = [&](int r, int c) -> int {
        if (r < 0 || c < 0 || r >= m || c >= n || grid[r][c] != 1) return 0;
        grid[r][c] = 0;
        return 1 + dfs(r + 1, c) + dfs(r - 1, c) + dfs(r, c + 1) + dfs(r, c - 1);
    };
    int best = 0;
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c)
            if (grid[r][c] == 1) best = std::max(best, dfs(r, c));
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{
        {0,0,1,0,0,0,0,1,0,0,0,0,0},
        {0,0,0,0,0,0,0,1,1,1,0,0,0},
        {0,1,1,0,1,0,0,0,0,0,0,0,0},
        {0,1,0,0,1,1,0,0,1,0,1,0,0},
        {0,1,0,0,1,1,0,0,1,1,1,0,0},
        {0,0,0,0,0,0,0,0,0,0,1,0,0},
        {0,0,0,0,0,0,0,1,1,1,0,0,0},
        {0,0,0,0,0,0,0,1,1,0,0,0,0}};
      if (maxAreaOfIsland(g) != 6) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{0,0,0},{0,0,0}};             if (maxAreaOfIsland(g) != 0) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{1}};                         if (maxAreaOfIsland(g) != 1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{1,1},{1,1}};                 if (maxAreaOfIsland(g) != 4) { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{1,0,1},{0,0,0},{1,0,1}};     if (maxAreaOfIsland(g) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Scan for land; each time you hit an unvisited `1`, flood-fill its component, sinking every visited cell to `0` and returning the cell count via `1 + the four recursive calls`. Track the maximum size seen. Every cell is processed once. O(m*n) time, O(m*n) worst-case stack space.
