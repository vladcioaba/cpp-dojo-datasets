## challenge: Maximal Square
tags: dynamic-programming, matrix
track: faang
difficulty: hard

Given an `m x n` binary matrix filled with the characters `'0'` and `'1'`, find the largest square that contains only `'1'`s and return its area.

Constraints: `1 <= m, n <= 300`, each cell is `'0'` or `'1'`.

Example: `matrix = [['1','0','1','0','0'],['1','0','1','1','1'],['1','1','1','1','1'],['1','0','0','1','0']]` → `4` (a 2x2 square of ones). Example: `matrix = [['0','1'],['1','0']]` → `1`.

hint: A cell can be the bottom-right corner of a square only if the cells above, to the left, and diagonally up-left also anchor squares.
hint: Let `dp[i][j]` be the side length of the largest all-ones square whose bottom-right corner is `(i, j)`.
hint: If the cell is `'1'`, `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`; the answer is the largest side squared.

```cpp
// starter
#include <vector>
int maximalSquare(std::vector<std::vector<char>>& matrix);
```

```cpp
int maximalSquare(std::vector<std::vector<char>>& matrix) {
    int m = (int)matrix.size();
    if (m == 0) return 0;
    int n = (int)matrix[0].size();
    std::vector<int> dp(n + 1, 0);
    int best = 0;
    for (int i = 1; i <= m; ++i) {
        int prev = 0;  // dp[i-1][j-1]
        for (int j = 1; j <= n; ++j) {
            int tmp = dp[j];
            if (matrix[i - 1][j - 1] == '1') {
                dp[j] = std::min({dp[j], dp[j - 1], prev}) + 1;
                best = std::max(best, dp[j]);
            } else {
                dp[j] = 0;
            }
            prev = tmp;
        }
    }
    return best * best;
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
    { vector<vector<char>> m{{'1','0','1','0','0'},{'1','0','1','1','1'},{'1','1','1','1','1'},{'1','0','0','1','0'}};
      if (maximalSquare(m) != 4) { std::puts("case1"); return 1; } }
    { vector<vector<char>> m{{'0','1'},{'1','0'}}; if (maximalSquare(m) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<char>> m{{'0'}};              if (maximalSquare(m) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<char>> m{{'1'}};              if (maximalSquare(m) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<char>> m{{'1','1'},{'1','1'}}; if (maximalSquare(m) != 4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** dp[i][j] is the side of the largest all-ones square ending with its bottom-right corner at cell (i, j). A one-cell can extend the smallest of the squares ending above, to the left, and up-left by one, so dp[i][j] = min of those three plus one. Rolling a single row with a saved diagonal tracks the best side in O(m*n) time and O(n) space; the area is that side squared.
