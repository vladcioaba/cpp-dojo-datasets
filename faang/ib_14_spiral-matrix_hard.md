## challenge: Spiral Matrix
tags: math, matrix, simulation
track: faang
difficulty: hard

Given an `m x n` matrix, return all of its elements in spiral order: start at the top-left corner and traverse right across the top row, down the right column, left across the bottom row, and up the left column, spiraling inward until every element has been visited.

Constraints: `m == matrix.length`, `n == matrix[0].length`, `1 <= m, n <= 10`, `-100 <= matrix[i][j] <= 100`.

Example: `matrix = [[1,2,3],[4,5,6],[7,8,9]]` → `[1,2,3,6,9,8,7,4,5]`. Example: `matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]` → `[1,2,3,4,8,12,11,10,9,5,6,7]`.

hint: Maintain four boundaries — top, bottom, left, right — and peel one edge at a time, shrinking the boundary you just consumed.
hint: Each outer loop pass walks right along `top`, down along `right`, left along `bottom`, and up along `left`, adjusting that boundary immediately after.
hint: After moving `top` down and `right` left, re-check `top <= bottom` and `left <= right` before the bottom and left passes so a thin leftover row or column is not visited twice.

```cpp
// starter
#include <vector>
std::vector<int> spiralOrder(std::vector<std::vector<int>>& matrix);
```

```cpp
std::vector<int> spiralOrder(std::vector<std::vector<int>>& matrix) {
    std::vector<int> res;
    if (matrix.empty()) return res;
    int top = 0, bottom = (int)matrix.size() - 1;
    int left = 0, right = (int)matrix[0].size() - 1;
    while (top <= bottom && left <= right) {
        for (int j = left; j <= right; ++j) res.push_back(matrix[top][j]);
        ++top;
        for (int i = top; i <= bottom; ++i) res.push_back(matrix[i][right]);
        --right;
        if (top <= bottom) {
            for (int j = right; j >= left; --j) res.push_back(matrix[bottom][j]);
            --bottom;
        }
        if (left <= right) {
            for (int i = bottom; i >= top; --i) res.push_back(matrix[i][left]);
            ++left;
        }
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> m{{1,2,3},{4,5,6},{7,8,9}};
      auto r = spiralOrder(m);
      if (r != vector<int>{1,2,3,6,9,8,7,4,5}) { std::puts("case1"); return 1; } }
    { vector<vector<int>> m{{1,2,3,4},{5,6,7,8},{9,10,11,12}};
      auto r = spiralOrder(m);
      if (r != vector<int>{1,2,3,4,8,12,11,10,9,5,6,7}) { std::puts("case2"); return 1; } }
    { vector<vector<int>> m{{1}};
      auto r = spiralOrder(m);
      if (r != vector<int>{1}) { std::puts("case3"); return 1; } }
    { vector<vector<int>> m{{1,2},{3,4}};
      auto r = spiralOrder(m);
      if (r != vector<int>{1,2,4,3}) { std::puts("case4"); return 1; } }
    { vector<vector<int>> m{{1},{2},{3},{4}};
      auto r = spiralOrder(m);
      if (r != vector<int>{1,2,3,4}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Track the four live edges of the not-yet-visited rectangle. Each iteration consumes the current top row left-to-right, the right column top-to-bottom, the bottom row right-to-left, and the left column bottom-to-top, tightening that boundary right after consuming it. The two guards `top <= bottom` and `left <= right` before the bottom and left passes prevent a single remaining row or column from being emitted a second time when the rectangle collapses to a line. Every cell is pushed exactly once, so the traversal is O(m·n) time and O(1) extra space beyond the output.
