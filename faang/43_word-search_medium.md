## challenge: Word Search
tags: backtracking, matrix, dfs
track: faang
difficulty: medium

Given an `m x n` grid of characters `board` and a string `word`, return `true` if `word` can be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighbouring (up/down/left/right). The same cell may not be used more than once.

Constraints: `1 <= m, n <= 6`, `1 <= word.length <= 15`, `board` and `word` consist of lowercase and uppercase English letters.

Example: `board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]]`, `word = "ABCCED"` -> `true`. Example: same board, `word = "ABCB"` -> `false`.

hint: This is a search over paths — from each starting cell, try to extend a match one adjacent letter at a time.
hint: Use DFS with backtracking: mark a cell visited before recursing into its four neighbours, then unmark it when you return so other paths can reuse it.
hint: Temporarily overwrite the current cell with a sentinel like `#` to block reuse, and restore the original character after the recursive calls.

```cpp
// starter
#include <vector>
#include <string>
bool exist(std::vector<std::vector<char>>& board, std::string word);
```

```cpp
bool exist(std::vector<std::vector<char>>& board, std::string word) {
    int m = (int)board.size();
    if (m == 0) return word.empty();
    int n = (int)board[0].size();
    std::function<bool(int, int, int)> dfs = [&](int r, int c, int k) -> bool {
        if (k == (int)word.size()) return true;
        if (r < 0 || c < 0 || r >= m || c >= n || board[r][c] != word[k]) return false;
        char saved = board[r][c];
        board[r][c] = '#';
        bool found = dfs(r + 1, c, k + 1) || dfs(r - 1, c, k + 1) ||
                     dfs(r, c + 1, k + 1) || dfs(r, c - 1, k + 1);
        board[r][c] = saved;
        return found;
    };
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c)
            if (dfs(r, c, 0)) return true;
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <functional>
using std::vector;
using std::string;
static vector<vector<char>> board(const vector<string>& rows) {
    vector<vector<char>> g;
    for (auto& r : rows) g.push_back(vector<char>(r.begin(), r.end()));
    return g;
}
//__USER__
int main() {
    { auto b = board({"ABCE","SFCS","ADEE"}); if (exist(b, "ABCCED") != true)  { std::puts("case1"); return 1; } }
    { auto b = board({"ABCE","SFCS","ADEE"}); if (exist(b, "SEE")    != true)  { std::puts("case2"); return 1; } }
    { auto b = board({"ABCE","SFCS","ADEE"}); if (exist(b, "ABCB")   != false) { std::puts("case3"); return 1; } }
    { auto b = board({"A"});                  if (exist(b, "A")      != true)  { std::puts("case4"); return 1; } }
    { auto b = board({"A"});                  if (exist(b, "B")      != false) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Try every cell as the path start and run DFS that matches `word` one character at a time. Mark the current cell as visited (overwrite with `#`) before recursing into the four neighbours and restore it on the way back so alternative paths remain valid. Each of the `m*n` starts explores up to 4 branches per letter, giving O(m*n*4^L) time for word length `L` and O(L) recursion depth.
