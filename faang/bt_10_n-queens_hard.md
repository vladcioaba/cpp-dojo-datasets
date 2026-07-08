## challenge: N-Queens II
tags: backtracking, bit-manipulation
track: faang
difficulty: hard

The n-queens puzzle places `n` queens on an `n x n` chessboard so that no two queens attack each other — no two share a row, a column, or a diagonal. Given an integer `n`, return the number of distinct solutions to the n-queens puzzle.

Constraints: `1 <= n <= 9`.

Example: `n = 1` -> `1`. Example: `n = 4` -> `2`. Example: `n = 8` -> `92`. Example: `n = 2` -> `0` and `n = 3` -> `0` (no placement works).

hint: Place exactly one queen per row, so a solution is an assignment of a column to each row; you only need to avoid column and diagonal clashes.
hint: Track occupied columns and both diagonal directions. Two cells share a `/` diagonal when `row + col` is equal, and a `\` diagonal when `row - col` is equal.
hint: Backtrack row by row: for each free column at the current row, mark its column and two diagonals, recurse to the next row, then unmark. Count a solution when all `n` rows are filled.

```cpp
// starter
int totalNQueens(int n);
```

```cpp
int totalNQueens(int n) {
    std::vector<char> col(n, 0), diag(2 * n, 0), anti(2 * n, 0);
    std::function<int(int)> dfs = [&](int r) -> int {
        if (r == n) return 1;
        int count = 0;
        for (int c = 0; c < n; ++c) {
            int d = r - c + n;   // "\" diagonal, shifted to be non-negative
            int a = r + c;       // "/" diagonal
            if (col[c] || diag[d] || anti[a]) continue;
            col[c] = diag[d] = anti[a] = 1;
            count += dfs(r + 1);
            col[c] = diag[d] = anti[a] = 0;
        }
        return count;
    };
    return dfs(0);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
#include <algorithm>
using std::vector;
// Brute-force reference: try every column assignment (permutation) and check diagonals.
static int reference(int n) {
    vector<int> p(n);
    for (int i = 0; i < n; ++i) p[i] = i;
    int cnt = 0;
    do {
        bool ok = true;
        for (int i = 0; i < n && ok; ++i)
            for (int j = i + 1; j < n; ++j)
                if (std::abs(p[i] - p[j]) == j - i) { ok = false; break; }
        if (ok) ++cnt;
    } while (std::next_permutation(p.begin(), p.end()));
    return cnt;
}
//__USER__
int main() {
    for (int n = 1; n <= 8; ++n) {
        if (totalNQueens(n) != reference(n)) { std::printf("n=%d\n", n); return 1; }
    }
    if (totalNQueens(1) != 1) { std::puts("k1"); return 1; }
    if (totalNQueens(2) != 0) { std::puts("k2"); return 1; }
    if (totalNQueens(3) != 0) { std::puts("k3"); return 1; }
    if (totalNQueens(4) != 2) { std::puts("k4"); return 1; }
    if (totalNQueens(8) != 92) { std::puts("k8"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Fix one queen per row so a candidate is a column-per-row assignment; this removes row conflicts automatically. Maintain three boolean sets for occupied columns, `\` diagonals (keyed by `row - col`, shifted non-negative), and `/` diagonals (keyed by `row + col`). Backtracking tries each free column in the current row, marks the three sets, recurses, and unmarks. The reference verifies against a permutation-based checker for small `n`. Time is bounded by the pruned search, well below the naive O(n!).
