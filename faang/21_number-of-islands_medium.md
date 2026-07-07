## challenge: Number of Islands
tags: graph, dfs, matrix, flood-fill
track: faang
difficulty: medium

Given an `m x n` grid of `'1'` (land) and `'0'` (water), return the number of islands. An island is land connected 4-directionally (up/down/left/right) and surrounded by water. You may mutate the grid.

Constraints: `1 <= m, n <= 300`, each cell is `'0'` or `'1'`.

Example: a grid with one big connected land mass → `1`. A grid with three separated clusters → `3`.

hint: Each time you find a piece of unvisited land it is a new island — then erase everything reachable from it so you never count it twice.
hint: Flood fill (DFS or BFS) from each land cell, sinking visited land to water as you go.

```cpp
// starter
#include <vector>
int numIslands(std::vector<std::vector<char>>& grid);
```

```cpp
int numIslands(std::vector<std::vector<char>>& grid) {
    int m = (int)grid.size();
    if (m == 0) return 0;
    int n = (int)grid[0].size();
    int count = 0;
    std::function<void(int, int)> dfs = [&](int r, int c) {
        if (r < 0 || c < 0 || r >= m || c >= n || grid[r][c] != '1') return;
        grid[r][c] = '0';
        dfs(r + 1, c); dfs(r - 1, c);
        dfs(r, c + 1); dfs(r, c - 1);
    };
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c)
            if (grid[r][c] == '1') { ++count; dfs(r, c); }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <functional>
using std::vector;
using std::string;
static vector<vector<char>> grid(const vector<string>& rows) {
    vector<vector<char>> g;
    for (auto& r : rows) g.push_back(vector<char>(r.begin(), r.end()));
    return g;
}
//__USER__
int main() {
    { auto g = grid({"11110","11010","11000","00000"}); if (numIslands(g) != 1) { std::puts("case1"); return 1; } }
    { auto g = grid({"11000","11000","00100","00011"}); if (numIslands(g) != 3) { std::puts("case2"); return 1; } }
    { auto g = grid({"000","000"});                     if (numIslands(g) != 0) { std::puts("case3"); return 1; } }
    { auto g = grid({"1"});                             if (numIslands(g) != 1) { std::puts("case4"); return 1; } }
    { auto g = grid({"101","010","101"});               if (numIslands(g) != 5) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Scan the grid; on hitting a '1', increment the island count and flood-fill all connected land to '0' so it is not recounted. Each cell is touched a constant number of times. O(m*n) time, O(m*n) worst-case stack/queue space.
