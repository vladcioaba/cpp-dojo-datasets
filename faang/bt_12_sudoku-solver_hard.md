## challenge: Sudoku Solver
tags: backtracking, hash-table, matrix
track: faang
difficulty: hard

Write a program to determine whether a `9 x 9` Sudoku board can be solved, filling it in place when it can. The board is given as a grid of characters where `'1'`-`'9'` are filled cells and `'.'` marks an empty cell. A valid solution requires that each of the digits `1`-`9` appears exactly once in every row, every column, and each of the nine `3 x 3` sub-boxes. Return `true` if the board has a solution and mutate `board` into that completed solution; return `false` if no solution exists (leaving the board in any state). The given clues are assumed consistent, and when a solution exists it is unique.

Constraints: `board.length == 9`, `board[i].length == 9`, each cell is a digit `'1'`-`'9'` or `'.'`.

Example: a standard puzzle with a valid completion returns `true` and the board becomes fully filled. Example: a board whose empty cells cannot all be filled legally returns `false`.

hint: Scan the cells in order; at the first empty cell try each digit `1`-`9` that does not already appear in its row, column, or `3 x 3` box, then recurse.
hint: If a chosen digit leads to a dead end, undo it (reset the cell to `'.'`) and try the next digit — standard backtracking with in-place mutation.
hint: The box containing cell `(r, c)` starts at row `3 * (r / 3)` and column `3 * (c / 3)`; return `true` as soon as every cell is filled, and `false` if no digit fits an empty cell.

```cpp
// starter
#include <vector>
bool solveSudoku(std::vector<std::vector<char>>& board);
```

```cpp
bool solveSudoku(std::vector<std::vector<char>>& board) {
    auto canPlace = [&](int r, int c, char v) {
        for (int i = 0; i < 9; ++i) {
            if (board[r][i] == v) return false;
            if (board[i][c] == v) return false;
            int br = 3 * (r / 3) + i / 3, bc = 3 * (c / 3) + i % 3;
            if (board[br][bc] == v) return false;
        }
        return true;
    };
    std::function<bool(int)> dfs = [&](int pos) -> bool {
        if (pos == 81) return true;
        int r = pos / 9, c = pos % 9;
        if (board[r][c] != '.') return dfs(pos + 1);
        for (char v = '1'; v <= '9'; ++v) {
            if (canPlace(r, c, v)) {
                board[r][c] = v;
                if (dfs(pos + 1)) return true;
                board[r][c] = '.';
            }
        }
        return false;
    };
    return dfs(0);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <functional>
#include <algorithm>
using std::vector;
using std::string;
static vector<vector<char>> mk(const vector<string>& rows) {
    vector<vector<char>> b(9, vector<char>(9));
    for (int i = 0; i < 9; ++i) for (int j = 0; j < 9; ++j) b[i][j] = rows[i][j];
    return b;
}
// Verify a completed board: every row, column, and 3x3 box holds digits 1..9 exactly once.
static bool fullyValid(const vector<vector<char>>& b) {
    const int all = 0b1111111110; // bits 1..9 set
    for (int i = 0; i < 9; ++i) {
        int row = 0, col = 0;
        for (int j = 0; j < 9; ++j) {
            if (b[i][j] < '1' || b[i][j] > '9') return false;
            row |= 1 << (b[i][j] - '0');
            col |= 1 << (b[j][i] - '0');
        }
        if (row != all || col != all) return false;
    }
    for (int br = 0; br < 9; br += 3)
        for (int bc = 0; bc < 9; bc += 3) {
            int box = 0;
            for (int i = 0; i < 3; ++i) for (int j = 0; j < 3; ++j) box |= 1 << (b[br + i][bc + j] - '0');
            if (box != all) return false;
        }
    return true;
}
//__USER__
int main() {
    {
        vector<string> rows = {
            "53..7....",
            "6..195...",
            ".98....6.",
            "8...6...3",
            "4..8.3..1",
            "7...2...6",
            ".6....28.",
            "...419..5",
            "....8..79"
        };
        auto given = mk(rows);
        auto board = given;
        bool ok = solveSudoku(board);
        if (!ok) { std::puts("solvable-returned-false"); return 1; }
        if (!fullyValid(board)) { std::puts("invalid-solution"); return 1; }
        // original clues must be preserved
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j)
                if (given[i][j] != '.' && board[i][j] != given[i][j]) { std::puts("clue-changed"); return 1; }
    }
    {
        // Unsolvable: row 0 already holds 1..8, so cell (0,8) must be 9,
        // but column 8 already contains a 9 at (1,8) -> no legal completion.
        vector<string> rows = {
            "12345678.",
            "........9",
            ".........",
            ".........",
            ".........",
            ".........",
            ".........",
            ".........",
            "........."
        };
        auto board = mk(rows);
        if (solveSudoku(board)) { std::puts("unsolvable-returned-true"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Walk the 81 cells in row-major order; skip filled cells and, at each empty one, try every digit that does not already occur in the cell's row, column, or `3 x 3` box, recursing on success and resetting to `'.'` on failure. Returning `true` at cell 81 means the board is complete, while an empty cell with no legal digit forces backtracking, and exhausting the first empty cell's options returns `false` for an unsolvable board. The search is exponential in the worst case but heavily pruned by the constraint checks; the board is solved in place.
