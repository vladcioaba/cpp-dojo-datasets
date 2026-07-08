## challenge: Pacific Atlantic Water Flow

tags: graph, dfs, matrix

track: faang
difficulty: hard

You are given an `m x n` matrix `heights` of non-negative integers representing the height of each cell of an island. The Pacific ocean borders the top and left edges; the Atlantic ocean borders the bottom and right edges. Water flows from a cell to any 4-directionally adjacent cell whose height is less than or equal to the current cell's height, and water can flow off any edge into its bordering ocean. Return a list of `[r, c]` coordinates of the cells from which water can reach **both** oceans (in any order).

Constraints: `1 <= m, n <= 200`, `0 <= heights[r][c] <= 10^5`.

Example: `heights = [[1,2,2,3,5],[3,2,3,4,4],[2,4,5,3,1],[6,7,1,4,5],[5,1,1,2,4]]` → `[[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]`. Example: `heights = [[1]]` → `[[0,0]]`.

hint: Simulating a flow from every cell is wasteful — instead ask which cells the ocean can reach if the water flowed uphill from the shore.
hint: From the Pacific border cells run a search that only steps to neighbours with height **greater than or equal** to the current cell, marking every reachable cell; do the same from the Atlantic border.
hint: The answer is the intersection: cells marked reachable from both the Pacific and the Atlantic searches.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> pacificAtlantic(std::vector<std::vector<int>>& heights);
```

```cpp
std::vector<std::vector<int>> pacificAtlantic(std::vector<std::vector<int>>& heights) {
    int m = (int)heights.size();
    if (m == 0) return {};
    int n = (int)heights[0].size();
    std::vector<std::vector<char>> pac(m, std::vector<char>(n, 0));
    std::vector<std::vector<char>> atl(m, std::vector<char>(n, 0));
    std::function<void(int, int, std::vector<std::vector<char>>&)> dfs =
        [&](int r, int c, std::vector<std::vector<char>>& vis) {
            vis[r][c] = 1;
            int dr[] = {1, -1, 0, 0}, dc[] = {0, 0, 1, -1};
            for (int d = 0; d < 4; ++d) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr < 0 || nc < 0 || nr >= m || nc >= n || vis[nr][nc]) continue;
                if (heights[nr][nc] < heights[r][c]) continue;
                dfs(nr, nc, vis);
            }
        };
    for (int r = 0; r < m; ++r) { dfs(r, 0, pac); dfs(r, n - 1, atl); }
    for (int c = 0; c < n; ++c) { dfs(0, c, pac); dfs(m - 1, c, atl); }
    std::vector<std::vector<int>> res;
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c)
            if (pac[r][c] && atl[r][c]) res.push_back({r, c});
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
#include <algorithm>
using std::vector;
static bool sameSet(vector<vector<int>> a, vector<vector<int>> b) {
    std::sort(a.begin(), a.end());
    std::sort(b.begin(), b.end());
    return a == b;
}
//__USER__
int main() {
    { vector<vector<int>> h{{1,2,2,3,5},{3,2,3,4,4},{2,4,5,3,1},{6,7,1,4,5},{5,1,1,2,4}};
      auto r = pacificAtlantic(h);
      vector<vector<int>> exp{{0,4},{1,3},{1,4},{2,2},{3,0},{3,1},{4,0}};
      if (!sameSet(r, exp)) { std::puts("case1"); return 1; } }
    { vector<vector<int>> h{{1}};       auto r = pacificAtlantic(h);
      vector<vector<int>> exp{{0,0}};
      if (!sameSet(r, exp)) { std::puts("case2"); return 1; } }
    { vector<vector<int>> h{{1,1},{1,1}}; auto r = pacificAtlantic(h);
      vector<vector<int>> exp{{0,0},{0,1},{1,0},{1,1}};
      if (!sameSet(r, exp)) { std::puts("case3"); return 1; } }
    { vector<vector<int>> h{{2,1},{1,2}}; auto r = pacificAtlantic(h);
      vector<vector<int>> exp{{0,0},{0,1},{1,0},{1,1}};
      if (!sameSet(r, exp)) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Reverse the flow. Instead of testing whether each cell can drain to an ocean, ask which cells an ocean can "climb" to: run a DFS/BFS inward from every Pacific border cell, stepping only to neighbours of height greater than or equal to the current one, and mark them; repeat from the Atlantic border. A cell drains to both oceans exactly when it is marked by both traversals, so the answer is their intersection. Each cell is visited O(1) times per ocean. O(m*n) time, O(m*n) space.
