## challenge: Unique Paths II
tags: dynamic-programming, array, matrix
track: faang
difficulty: medium

A robot starts in the top-left cell of an `m x n` grid and wants to reach the bottom-right cell, moving only right or down. Some cells contain an obstacle marked `1` (empty cells are `0`), and the robot cannot enter an obstacle. Return the number of distinct paths to the destination.

Constraints: `1 <= m, n <= 100`, each cell is `0` or `1`. The answer is guaranteed to fit in a 32-bit signed integer.

Example: `obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]` → `2`. Example: `obstacleGrid = [[0,1],[0,0]]` → `1`.

hint: An obstacle cell contributes zero paths; treat it as unreachable.
hint: For an open cell, the number of paths is the sum coming from the left and from above.

```cpp
// starter
#include <vector>
int uniquePathsWithObstacles(std::vector<std::vector<int>>& obstacleGrid);
```

```cpp
int uniquePathsWithObstacles(std::vector<std::vector<int>>& g) {
    int m = g.size(), n = g[0].size();
    std::vector<long long> dp(n, 0);
    dp[0] = (g[0][0] == 0) ? 1 : 0;
    for (int i = 0; i < m; ++i)
        for (int j = 0; j < n; ++j) {
            if (g[i][j] == 1) dp[j] = 0;
            else if (j > 0) dp[j] += dp[j - 1];
        }
    return (int)dp[n - 1];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{{0,0,0},{0,1,0},{0,0,0}}; if (uniquePathsWithObstacles(g) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{0,1},{0,0}};             if (uniquePathsWithObstacles(g) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{1}};                     if (uniquePathsWithObstacles(g) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{0}};                     if (uniquePathsWithObstacles(g) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{0,0},{1,1},{0,0}};       if (uniquePathsWithObstacles(g) != 0) { std::puts("case5"); return 1; } }
    { vector<vector<int>> g{{0,0,0,0},{0,0,0,0},{0,0,0,0}}; if (uniquePathsWithObstacles(g) != 10) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[i][j]` count paths reaching cell `(i,j)`. An obstacle sets that count to zero; otherwise the cell inherits `dp[i-1][j] + dp[i][j-1]`, the paths arriving from above and from the left. Sweeping row by row, one length-`n` array holds the previous row so that `dp[j]` (from above) and `dp[j-1]` (from the left, already updated this row) combine in place. Counts are accumulated in 64-bit to avoid intermediate overflow. O(m·n) time, O(n) space.
