## challenge: Path With Minimum Effort
tags: graph, dijkstra, matrix, binary-search
track: faang
difficulty: hard

You are a hiker preparing for a trip. You are given `heights`, a 2D array of size `rows x columns`, where `heights[r][c]` is the height of the cell `(r, c)`. You start at the top-left cell `(0, 0)` and hope to travel to the bottom-right cell `(rows-1, columns-1)`, moving up, down, left, or right. The effort of a route is the maximum absolute difference in heights between two consecutive cells along it. Return the minimum effort required to travel from the top-left to the bottom-right cell.

Constraints: `rows == heights.length`, `columns == heights[0].length`, `1 <= rows, columns <= 100`, `1 <= heights[r][c] <= 10^6`.

Example: `heights = [[1,2,2],[3,8,2],[5,3,5]]` → `2` (the route 1→2→2→2→5→3 has max adjacent difference 2). Example: `heights = [[1,2,3],[3,8,4],[5,3,5]]` → `1`.

hint: Instead of summing weights, the cost of a path is the maximum edge weight along it. Adapt shortest-path search to minimize a maximum.
hint: Run Dijkstra where a cell's cost is the minimum over paths of the maximum adjacent difference; relax with `new = max(current, |Δheight|)`. The first time you pop the destination you have the answer.

```cpp
// starter
#include <vector>
int minimumEffortPath(std::vector<std::vector<int>>& heights);
```

```cpp
#include <queue>
#include <tuple>
#include <climits>
#include <cstdlib>
#include <algorithm>
int minimumEffortPath(std::vector<std::vector<int>>& heights) {
    int m = (int)heights.size(), n = (int)heights[0].size();
    std::vector<std::vector<int>> effort(m, std::vector<int>(n, INT_MAX));
    effort[0][0] = 0;
    std::priority_queue<std::tuple<int, int, int>, std::vector<std::tuple<int, int, int>>,
                        std::greater<std::tuple<int, int, int>>> pq;
    pq.push({0, 0, 0});
    int dr[] = {0, 0, 1, -1}, dc[] = {1, -1, 0, 0};
    while (!pq.empty()) {
        auto [e, r, c] = pq.top(); pq.pop();
        if (r == m - 1 && c == n - 1) return e;
        if (e > effort[r][c]) continue;
        for (int d = 0; d < 4; ++d) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            int ne = std::max(e, std::abs(heights[nr][nc] - heights[r][c]));
            if (ne < effort[nr][nc]) { effort[nr][nc] = ne; pq.push({ne, nr, nc}); }
        }
    }
    return 0;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> h{{1,2,2},{3,8,2},{5,3,5}}; if (minimumEffortPath(h) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> h{{1,2,3},{3,8,4},{5,3,5}}; if (minimumEffortPath(h) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> h{{1,2,1,1,1},{1,2,1,2,1},{1,2,1,2,1},{1,2,1,2,1},{1,1,1,2,1}}; if (minimumEffortPath(h) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<int>> h{{1}}; if (minimumEffortPath(h) != 0) { std::puts("case4"); return 1; } }
    { vector<vector<int>> h{{1,10,6,7,9,10,4,9}}; if (minimumEffortPath(h) != 9) { std::puts("case5"); return 1; } }
    { vector<vector<int>> h{{3},{8},{4}}; if (minimumEffortPath(h) != 5) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The cost of a route is the maximum adjacent height difference along it, and we want to minimize that maximum — a minimax path problem. Dijkstra adapts naturally: keep `effort[r][c]` as the best (smallest) achievable route-maximum to that cell, using a min-heap. Relaxing an edge sets the candidate cost to `max(currentEffort, |heightDiff|)`. Because heap pops occur in nondecreasing order of effort, the first time the destination is popped its value is optimal. Time is O(m * n * log(m * n)), space O(m * n). (A binary search over the answer combined with BFS is an alternative.)
