## challenge: Search a 2D Matrix II
tags: binary-search, matrix
track: faang
difficulty: medium

Write an efficient algorithm that searches for a `target` in an `m x n` integer matrix. Each **row** is sorted in ascending order left to right, and each **column** is sorted in ascending order top to bottom. Return `true` if `target` is present, `false` otherwise.

Constraints: `1 <= m, n <= 300`, `-10^9 <= matrix[i][j], target <= 10^9`, each row and each column is sorted ascending.

Example: `matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 5` → `true`. Example: `target = 20` → `false`. Example: `matrix = [[1]], target = 1` → `true`.

hint: Unlike a fully sorted matrix, you cannot flatten this one — but each step can still eliminate a whole row or column.
hint: Start at the top-right corner: it is the largest in its row and the smallest in its column, so one comparison decides which to drop.
hint: If the corner value is greater than `target`, move left (drop that column); if it is smaller, move down (drop that row).

```cpp
// starter
#include <vector>
bool searchMatrix(std::vector<std::vector<int>>& matrix, int target);
```

```cpp
bool searchMatrix(std::vector<std::vector<int>>& matrix, int target) {
    int m = (int)matrix.size(), n = (int)matrix[0].size();
    int r = 0, c = n - 1;             // start at the top-right corner
    while (r < m && c >= 0) {
        int v = matrix[r][c];
        if (v == target) return true;
        if (v > target) --c;          // too big: this column is all too big below
        else ++r;                     // too small: this row is all too small left
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    vector<vector<int>> mtx{{1,4,7,11,15},{2,5,8,12,19},{3,6,9,16,22},{10,13,14,17,24},{18,21,23,26,30}};
    if (searchMatrix(mtx, 5)  != true)  { std::puts("case1"); return 1; }
    if (searchMatrix(mtx, 20) != false) { std::puts("case2"); return 1; }
    if (searchMatrix(mtx, 1)  != true)  { std::puts("case3"); return 1; }
    if (searchMatrix(mtx, 30) != true)  { std::puts("case4"); return 1; }
    if (searchMatrix(mtx, 0)  != false) { std::puts("case5"); return 1; }
    { vector<vector<int>> one{{1}};
      if (searchMatrix(one, 1) != true)  { std::puts("case6"); return 1; }
      if (searchMatrix(one, 2) != false) { std::puts("case7"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The top-right corner is simultaneously the maximum of its row and the minimum of its column, so comparing it with `target` unambiguously eliminates either the current column (move left) or the current row (move down). Each step removes one row or one column, giving an O(m + n) staircase search with O(1) space — strictly better than binary-searching every row. The bottom-left corner works symmetrically.
