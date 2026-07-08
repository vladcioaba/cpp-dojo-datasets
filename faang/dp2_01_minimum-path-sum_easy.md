## challenge: Minimum Path Sum
tags: dynamic-programming, array, matrix
track: faang
difficulty: easy

Given an `m x n` grid filled with non-negative numbers, find a path from the top-left cell to the bottom-right cell that minimizes the sum of the numbers along the path. You may only move either down or right at any step. Return that minimum sum.

Constraints: `1 <= m, n <= 200`, `0 <= grid[i][j] <= 200`.

Example: `grid = [[1,3,1],[1,5,1],[4,2,1]]` → `7` (the path `1 -> 3 -> 1 -> 1 -> 1`). Example: `grid = [[1,2,3],[4,5,6]]` → `12`.

hint: The cheapest way into a cell arrives from either its left neighbor or its top neighbor.
hint: `dp[j] = grid[i][j] + min(dp[j], dp[j-1])` lets one rolling row replace the full table.

```cpp
// starter
#include <vector>
int minPathSum(std::vector<std::vector<int>>& grid);
```

```cpp
int minPathSum(std::vector<std::vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    std::vector<int> dp(n, 0);
    dp[0] = grid[0][0];
    for (int j = 1; j < n; ++j) dp[j] = dp[j-1] + grid[0][j];
    for (int i = 1; i < m; ++i) {
        dp[0] += grid[i][0];
        for (int j = 1; j < n; ++j)
            dp[j] = std::min(dp[j-1], dp[j]) + grid[i][j];
    }
    return dp[n-1];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{{1,3,1},{1,5,1},{4,2,1}}; if (minPathSum(g) != 7)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{1,2,3},{4,5,6}};         if (minPathSum(g) != 12) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{5}};                     if (minPathSum(g) != 5)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{1,2},{1,1}};             if (minPathSum(g) != 3)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{1,2,5},{3,2,1}};         if (minPathSum(g) != 6)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[i][j]` be the minimum path sum to reach cell `(i,j)`. Since moves are only right or down, that cell is entered from the left or from above, giving `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`. Processing rows top to bottom lets a single length-`n` array carry the previous row; before overwriting column `j` it still holds the value from above, and `dp[j-1]` is the value from the left. O(m·n) time, O(n) space.
