## challenge: Surrounded Regions

tags: graph, dfs, matrix

track: faang
difficulty: medium

You are given an `m x n` board of characters `'X'` and `'O'`. Capture every region of `'O'`s that is completely surrounded by `'X'`s by flipping all of that region's `'O'`s to `'X'`. A region is *not* captured if any of its cells touches the border of the board (directly or through connected `'O'`s). Modify the board in place.

Constraints: `1 <= m, n <= 200`, each cell is `'X'` or `'O'`.

Example: `[["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]]` → `[["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]]` (only the border-touching `'O'` at the bottom survives).

hint: It is easier to find the `'O'`s that *survive* than the ones that get captured — survivors are exactly those connected to the border.
hint: Flood fill from every `'O'` on the four edges, marking each reachable `'O'` as safe (e.g. temporarily `'#'`).
hint: After marking, sweep the board: any remaining `'O'` was surrounded (flip to `'X'`), and every `'#'` becomes `'O'` again.

```cpp
// starter
#include <vector>
void solve(std::vector<std::vector<char>>& board);
```

```cpp
void solve(std::vector<std::vector<char>>& board) {
    int m = (int)board.size();
    if (m == 0) return;
    int n = (int)board[0].size();
    std::function<void(int, int)> dfs = [&](int r, int c) {
        if (r < 0 || c < 0 || r >= m || c >= n || board[r][c] != 'O') return;
        board[r][c] = '#';
        dfs(r + 1, c); dfs(r - 1, c);
        dfs(r, c + 1); dfs(r, c - 1);
    };
    for (int r = 0; r < m; ++r) { dfs(r, 0); dfs(r, n - 1); }
    for (int c = 0; c < n; ++c) { dfs(0, c); dfs(m - 1, c); }
    for (int r = 0; r < m; ++r)
        for (int c = 0; c < n; ++c) {
            if (board[r][c] == 'O') board[r][c] = 'X';
            else if (board[r][c] == '#') board[r][c] = 'O';
        }
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
static vector<vector<char>> grid(const vector<string>& rows) {
    vector<vector<char>> g;
    for (auto& r : rows) g.push_back(vector<char>(r.begin(), r.end()));
    return g;
}
static bool eq(const vector<vector<char>>& g, const vector<string>& rows) {
    if (g.size() != rows.size()) return false;
    for (int r = 0; r < (int)g.size(); ++r) {
        if (g[r].size() != rows[r].size()) return false;
        for (int c = 0; c < (int)g[r].size(); ++c) if (g[r][c] != rows[r][c]) return false;
    }
    return true;
}
//__USER__
int main() {
    { auto g = grid({"XXXX","XOOX","XXOX","XOXX"}); solve(g); if (!eq(g, {"XXXX","XXXX","XXXX","XOXX"})) { std::puts("case1"); return 1; } }
    { auto g = grid({"X"});                         solve(g); if (!eq(g, {"X"}))                         { std::puts("case2"); return 1; } }
    { auto g = grid({"OOO","OOO","OOO"});           solve(g); if (!eq(g, {"OOO","OOO","OOO"}))           { std::puts("case3"); return 1; } }
    { auto g = grid({"XXX","XOX","XXX"});           solve(g); if (!eq(g, {"XXX","XXX","XXX"}))           { std::puts("case4"); return 1; } }
    { auto g = grid({"XOX","OXO","XOX"});           solve(g); if (!eq(g, {"XOX","OXO","XOX"}))           { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Invert the problem: an `'O'` is captured unless it can reach the border through connected `'O'`s. Flood-fill from every border `'O'`, marking survivors with a sentinel `'#'`. Then a single pass flips leftover `'O'`s (truly surrounded) to `'X'` and restores each `'#'` to `'O'`. Each cell is visited a constant number of times. O(m*n) time, O(m*n) worst-case stack space.
