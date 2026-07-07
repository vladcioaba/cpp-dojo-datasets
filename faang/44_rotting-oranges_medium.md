## challenge: Rotting Oranges
tags: bfs, matrix, graph
track: faang
difficulty: medium

You are given an `m x n` grid where each cell is `0` (empty), `1` (a fresh orange), or `2` (a rotten orange). Every minute, any fresh orange that is 4-directionally adjacent to a rotten orange becomes rotten. Return the minimum number of minutes that must elapse until no cell has a fresh orange, or `-1` if that is impossible.

Constraints: `1 <= m, n <= 10`, each `grid[i][j]` is `0`, `1`, or `2`.

Example: `grid = [[2,1,1],[1,1,0],[0,1,1]]` -> `4`. Example: `grid = [[2,1,1],[0,1,1],[1,0,1]]` -> `-1` (the bottom-left orange is unreachable).

hint: Rot spreads outward in waves from every rotten orange at once — this is a shortest-time-to-reach problem, so think breadth-first.
hint: Seed a queue with all initially rotten oranges (multi-source BFS) and count fresh oranges up front.
hint: Process the queue one full level per minute; if any fresh oranges remain after BFS finishes, return `-1`.

```cpp
// starter
#include <vector>
int orangesRotting(std::vector<std::vector<int>>& grid);
```

```cpp
int orangesRotting(std::vector<std::vector<int>>& grid) {
    int m = (int)grid.size();
    if (m == 0) return 0;
    int n = (int)grid[0].size();
    std::queue<std::pair<int, int>> q;
    int fresh = 0;
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c) {
            if (grid[r][c] == 2) q.push({r, c});
            else if (grid[r][c] == 1) ++fresh;
        }
    if (fresh == 0) return 0;
    int minutes = 0;
    int dr[] = {1, -1, 0, 0}, dc[] = {0, 0, 1, -1};
    while (!q.empty() && fresh > 0) {
        int sz = (int)q.size();
        for (int i = 0; i < sz; ++i) {
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; ++d) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr < 0 || nc < 0 || nr >= m || nc >= n || grid[nr][nc] != 1) continue;
                grid[nr][nc] = 2;
                --fresh;
                q.push({nr, nc});
            }
        }
        ++minutes;
    }
    return fresh == 0 ? minutes : -1;
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
    { vector<vector<int>> g{{2,1,1},{1,1,0},{0,1,1}}; if (orangesRotting(g) != 4)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{2,1,1},{0,1,1},{1,0,1}}; if (orangesRotting(g) != -1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{0,2}};                   if (orangesRotting(g) != 0)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{0,0},{0,0}};             if (orangesRotting(g) != 0)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{1}};                     if (orangesRotting(g) != -1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Run a multi-source BFS starting from every rotten orange simultaneously, tracking the count of fresh oranges. Each BFS level corresponds to one minute; a fresh neighbour becomes rotten and is enqueued. When the queue drains, the elapsed levels give the answer, unless fresh oranges remain (unreachable), in which case return `-1`. O(m*n) time and space since each cell is enqueued at most once.
