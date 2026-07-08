## challenge: Unique Paths
tags: dynamic-programming, math
track: faang
difficulty: easy

A robot sits in the top-left cell of an `m x n` grid and wants to reach the bottom-right cell. At each move it can step only right or down. Return the number of distinct paths it can take.

Constraints: `1 <= m, n <= 100`. The answer is guaranteed to fit in a 32-bit signed integer.

Example: `m = 3, n = 7` → `28`. Example: `m = 3, n = 2` → `3` (down-down-right, down-right-down, right-down-down).

hint: The only ways into a cell are from its left neighbor or its top neighbor.
hint: Let `dp[i][j]` be the number of paths reaching cell `(i, j)`; the first row and column each have exactly one path.
hint: `dp[i][j] = dp[i-1][j] + dp[i][j-1]`, which collapses to a single rolling row.

```cpp
// starter
int uniquePaths(int m, int n);
```

```cpp
int uniquePaths(int m, int n) {
    std::vector<int> dp(n, 1);
    for (int i = 1; i < m; ++i)
        for (int j = 1; j < n; ++j)
            dp[j] += dp[j - 1];
    return dp[n - 1];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    if (uniquePaths(3, 7) != 28) { std::puts("case1"); return 1; }
    if (uniquePaths(3, 2) != 3)  { std::puts("case2"); return 1; }
    if (uniquePaths(1, 1) != 1)  { std::puts("case3"); return 1; }
    if (uniquePaths(3, 3) != 6)  { std::puts("case4"); return 1; }
    if (uniquePaths(7, 3) != 28) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Each cell is reachable only from above or from the left, so dp[i][j] = dp[i-1][j] + dp[i][j-1] with the top row and left column seeded to 1. Sweeping row by row, a single length-n array can be updated in place because dp[j] already holds the value from the row above before it is overwritten. O(m*n) time, O(n) space.
