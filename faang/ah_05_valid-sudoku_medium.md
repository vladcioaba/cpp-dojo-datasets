## challenge: Valid Sudoku
tags: array, hash-table, matrix, arrays-hashing
track: faang
difficulty: medium

Determine whether a partially filled `9 x 9` Sudoku board is valid. Only the currently placed digits must be checked against three rules: each row, each column, and each of the nine `3 x 3` sub-boxes may not contain a repeated digit `1`-`9`. Empty cells hold the character `'.'`. The board does not need to be solvable — only internally consistent.

Constraints: `board.length == board[i].length == 9`, each cell is a digit `'1'`-`'9'` or `'.'`.

Example: a standard partially filled board with no conflicts → `true`. Example: the same board with a second `8` placed in a column that already holds an `8` → `false`.

hint: A cell participates in exactly one row, one column, and one 3x3 box; a digit is illegal only if it collides in one of those three groups.
hint: Track "seen" digits for all 9 rows, 9 columns, and 9 boxes — nine small bitsets or boolean tables do the job.
hint: Map a cell at `(r, c)` to its box index with `(r / 3) * 3 + c / 3`, then reject the first time any of the three groups reports the digit already present.

```cpp
// starter
#include <vector>
bool isValidSudoku(std::vector<std::vector<char>>& board);
```

```cpp
bool isValidSudoku(std::vector<std::vector<char>>& board) {
    bool rows[9][9] = {}, cols[9][9] = {}, boxes[9][9] = {};
    for (int r = 0; r < 9; ++r) {
        for (int c = 0; c < 9; ++c) {
            char ch = board[r][c];
            if (ch == '.') continue;
            int d = ch - '1';
            int b = (r / 3) * 3 + c / 3;
            if (rows[r][d] || cols[c][d] || boxes[b][d]) return false;
            rows[r][d] = cols[c][d] = boxes[b][d] = true;
        }
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
using std::vector;
using std::string;
//__USER__
static vector<vector<char>> mk(vector<string> rows) {
    vector<vector<char>> b;
    for (auto& s : rows) b.emplace_back(s.begin(), s.end());
    return b;
}
int main() {
    auto valid = mk({
        "53..7....","6..195...",".98....6.",
        "8...6...3","4..8.3..1","7...2...6",
        ".6....28.","...419..5","....8..79"});
    if (!isValidSudoku(valid)) { std::puts("case1"); return 1; }

    auto colDup = mk({
        "83..7....","6..195...",".98....6.",
        "8...6...3","4..8.3..1","7...2...6",
        ".6....28.","...419..5","....8..79"});
    if (isValidSudoku(colDup)) { std::puts("case2"); return 1; }

    auto boxDup = mk({
        "53..7....","6..195...","398....6.",
        "8...6...3","4..8.3..1","7...2...6",
        ".6....28.","...419..5","....8..79"});
    if (isValidSudoku(boxDup)) { std::puts("case3"); return 1; }

    auto empty = mk({
        ".........",".........",".........",
        ".........",".........",".........",
        ".........",".........","........."});
    if (!isValidSudoku(empty)) { std::puts("case4"); return 1; }

    auto rowDup = mk({
        "55..7....","6..195...",".98....6.",
        "8...6...3","4..8.3..1","7...2...6",
        ".6....28.","...419..5","....8..79"});
    if (isValidSudoku(rowDup)) { std::puts("case5"); return 1; }

    std::puts("PASS");
}
```

**Editorial:** Each placed digit belongs to one row, one column, and one 3x3 box, so we keep a boolean "seen" table for all nine of each. As we scan every cell, the box index is `(r/3)*3 + c/3`; the first time a digit is already marked in its row, column, or box we return false. A single pass over the 81 cells gives O(1) time and O(1) space for this fixed-size board.
