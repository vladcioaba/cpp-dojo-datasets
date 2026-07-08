## challenge: Search a 2D Matrix
tags: binary-search, matrix
track: faang
difficulty: medium

You are given an `m x n` integer matrix with two properties: each row is sorted in ascending order left to right, and the first integer of each row is greater than the last integer of the previous row. Given a `target`, return `true` if it appears in the matrix. You must run in O(log(m*n)).

Constraints: `1 <= m, n <= 100`, `-10^4 <= matrix[i][j], target <= 10^4`.

Example: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3` → `true`. Example: `target = 13` → `false`. Example: `matrix = [[1]], target = 1` → `true`.

hint: The two properties together mean that reading the matrix row by row yields one fully sorted sequence.
hint: Treat the grid as a virtual array of length `m * n` and binary search over the flat index range.
hint: Convert a flat index `k` back to a cell with `matrix[k / n][k % n]` — no need to actually flatten the data.

```cpp
// starter
#include <vector>
bool searchMatrix(std::vector<std::vector<int>>& matrix, int target);
```

```cpp
bool searchMatrix(std::vector<std::vector<int>>& matrix, int target) {
    int m = (int)matrix.size(), n = (int)matrix[0].size();
    int lo = 0, hi = m * n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];
        if (val == target) return true;
        if (val < target) lo = mid + 1;
        else hi = mid - 1;
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
    { vector<vector<int>> mtx{{1,3,5,7},{10,11,16,20},{23,30,34,60}};
      if (searchMatrix(mtx, 3)  != true)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> mtx{{1,3,5,7},{10,11,16,20},{23,30,34,60}};
      if (searchMatrix(mtx, 13) != false) { std::puts("case2"); return 1; } }
    { vector<vector<int>> mtx{{1}};       if (searchMatrix(mtx, 1) != true)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> mtx{{1}};       if (searchMatrix(mtx, 0) != false) { std::puts("case4"); return 1; } }
    { vector<vector<int>> mtx{{1,1}};     if (searchMatrix(mtx, 1) != true)  { std::puts("case5"); return 1; } }
    { vector<vector<int>> mtx{{1,3,5,7},{10,11,16,20},{23,30,34,60}};
      if (searchMatrix(mtx, 60) != true)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because each row's minimum exceeds the previous row's maximum, the whole matrix is a single sorted sequence. Binary search over the index range `[0, m*n - 1]`, mapping a flat index `k` to the cell `(k / n, k % n)`. This gives O(log(m*n)) time and O(1) space without materializing the flattened array.
