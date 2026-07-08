## challenge: Triangle
tags: dynamic-programming, array
track: faang
difficulty: easy

Given a `triangle` array, return the minimum path sum from the top to the bottom. Each step you may move to an adjacent number on the row below: from index `j` on one row you may move to index `j` or `j+1` on the next row.

Constraints: `1 <= triangle.length <= 200`, `triangle[i].length == i + 1`, `-10^4 <= triangle[i][j] <= 10^4`.

Example: `triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]` → `11` (the path `2 -> 3 -> 5 -> 1`). Example: `triangle = [[-10]]` → `-10`.

hint: Work from the bottom row upward so each cell already knows the best of its two children.
hint: `dp[j] = triangle[i][j] + min(dp[j], dp[j+1])` collapses the two rows into one array.

```cpp
// starter
#include <vector>
int minimumTotal(std::vector<std::vector<int>>& triangle);
```

```cpp
int minimumTotal(std::vector<std::vector<int>>& triangle) {
    std::vector<int> dp(triangle.back());
    for (int i = (int)triangle.size() - 2; i >= 0; --i)
        for (int j = 0; j <= i; ++j)
            dp[j] = triangle[i][j] + std::min(dp[j], dp[j+1]);
    return dp[0];
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
    { vector<vector<int>> t{{2},{3,4},{6,5,7},{4,1,8,3}}; if (minimumTotal(t) != 11) { std::puts("case1"); return 1; } }
    { vector<vector<int>> t{{-10}};                       if (minimumTotal(t) != -10){ std::puts("case2"); return 1; } }
    { vector<vector<int>> t{{1},{2,3}};                   if (minimumTotal(t) != 3)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> t{{-1},{2,3},{1,-1,-3}};        if (minimumTotal(t) != -1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> t{{1},{1,1},{1,1,1}};           if (minimumTotal(t) != 3)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Define `dp[i][j]` as the minimum sum to descend from cell `(i,j)` to the bottom. A leaf's value is itself, and internally `dp[i][j] = triangle[i][j] + min(dp[i+1][j], dp[i+1][j+1])`. Filling from the bottom row up, one array of length equal to the base suffices: overwriting `dp[j]` in place uses the still-present children `dp[j]` and `dp[j+1]`. The answer is `dp[0]`. O(n^2) time in the number of rows, O(n) space.
