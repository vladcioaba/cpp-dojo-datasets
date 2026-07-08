## challenge: Shortest Path in Binary Matrix

tags: graph, bfs, matrix

track: faang
difficulty: hard

Given an `n x n` binary grid, a clear path from the top-left cell `(0, 0)` to the bottom-right cell `(n-1, n-1)` is a sequence of cells, all valued `0`, in which consecutive cells are 8-directionally connected (horizontally, vertically, or diagonally adjacent). Return the length of the shortest clear path (the number of cells it visits), or `-1` if no clear path exists.

Constraints: `1 <= n <= 100`, each `grid[r][c]` is `0` or `1`.

Example: `grid = [[0,0,0],[1,1,0],[1,1,0]]` → `4`. Example: `grid = [[0,1],[1,0]]` → `2`. Example: if `grid[0][0]` or `grid[n-1][n-1]` is `1`, return `-1`.

hint: Every legal move has the same cost, so the fewest-cells path is a shortest path in an unweighted graph — BFS finds it.
hint: A cell has up to eight neighbours here; enqueue the start (if it is `0`) and expand outward level by level.
hint: Record each cell's distance (or mark it visited) when you first reach it so BFS never re-processes a cell, and stop as soon as you pop the destination.

```cpp
// starter
#include <vector>
int shortestPathBinaryMatrix(std::vector<std::vector<int>>& grid);
```

```cpp
int shortestPathBinaryMatrix(std::vector<std::vector<int>>& grid) {
    int n = (int)grid.size();
    if (grid[0][0] != 0 || grid[n - 1][n - 1] != 0) return -1;
    std::vector<std::vector<int>> dist(n, std::vector<int>(n, -1));
    std::queue<std::pair<int, int>> q;
    q.push({0, 0});
    dist[0][0] = 1;
    int dr[] = {1, -1, 0, 0, 1, 1, -1, -1};
    int dc[] = {0, 0, 1, -1, 1, -1, 1, -1};
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        if (r == n - 1 && c == n - 1) return dist[r][c];
        for (int d = 0; d < 8; ++d) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nc < 0 || nr >= n || nc >= n) continue;
            if (grid[nr][nc] != 0 || dist[nr][nc] != -1) continue;
            dist[nr][nc] = dist[r][c] + 1;
            q.push({nr, nc});
        }
    }
    return -1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <utility>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{{0,1},{1,0}};                 if (shortestPathBinaryMatrix(g) != 2)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{0,0,0},{1,1,0},{1,1,0}};     if (shortestPathBinaryMatrix(g) != 4)  { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{1,0,0},{1,1,0},{1,1,0}};     if (shortestPathBinaryMatrix(g) != -1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{0}};                         if (shortestPathBinaryMatrix(g) != 1)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{0,1,0},{1,1,0},{0,1,0}};     if (shortestPathBinaryMatrix(g) != -1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** All moves cost one, so the minimum-cell path is a BFS shortest path over the 8-connected grid. Bail out immediately if either endpoint is blocked. Otherwise seed the queue at `(0,0)` with distance `1`, and expand each cell to its eight neighbours, assigning a distance and enqueuing the first time each open cell is reached. The distance recorded when the destination is popped is the answer; an exhausted queue means it is unreachable. O(n^2) time, O(n^2) space.
