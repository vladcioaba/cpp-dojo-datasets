## challenge: Generate Parentheses
tags: backtracking, string, dynamic-programming
track: faang
difficulty: easy

Given `n` pairs of parentheses, return all combinations of well-formed (balanced) parentheses using exactly `n` opening and `n` closing brackets. Return the answer in any order.

Constraints: `1 <= n <= 8`.

Example: `n = 1` -> `["()"]`. Example: `n = 3` -> `["((()))","(()())","(())()","()(())","()()()"]`. Example: `n = 2` -> `["(())","()()"]`.

hint: A string of length `2n` is well-formed exactly when no prefix has more `)` than `(`, and the totals of each are equal.
hint: Backtrack over positions, tracking how many `(` and `)` you have placed; you may add `(` while `open < n`, and `)` only while `close < open`.
hint: Record the string when its length reaches `2n` — every path that respects the two rules ends at a valid arrangement.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> generateParenthesis(int n);
```

```cpp
std::vector<std::string> generateParenthesis(int n) {
    std::vector<std::string> res;
    std::string cur;
    std::function<void(int, int)> dfs = [&](int open, int close) {
        if ((int)cur.size() == 2 * n) { res.push_back(cur); return; }
        if (open < n) { cur.push_back('('); dfs(open + 1, close); cur.pop_back(); }
        if (close < open) { cur.push_back(')'); dfs(open, close + 1); cur.pop_back(); }
    };
    dfs(0, 0);
    return res;
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
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
static bool balanced(const string& s) {
    int bal = 0;
    for (char c : s) { bal += (c == '(') ? 1 : -1; if (bal < 0) return false; }
    return bal == 0;
}
//__USER__
int main() {
    {
        auto got = generateParenthesis(1);
        vector<string> want = {"()"};
        if (canon(got) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        auto got = generateParenthesis(3);
        vector<string> want = {"((()))","(()())","(())()","()(())","()()()"};
        if (canon(got) != canon(want)) { std::puts("case2"); return 1; }
    }
    {
        auto got = generateParenthesis(2);
        vector<string> want = {"(())","()()"};
        if (canon(got) != canon(want)) { std::puts("case3"); return 1; }
    }
    {
        // Catalan(4) = 14, all well-formed and distinct
        auto got = generateParenthesis(4);
        if (got.size() != 14) { std::puts("size4"); return 1; }
        auto c = canon(got);
        c.erase(std::unique(c.begin(), c.end()), c.end());
        if (c.size() != 14) { std::puts("dup4"); return 1; }
        for (auto& s : got) if ((int)s.size() != 8 || !balanced(s)) { std::puts("bad4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Grow the string one bracket at a time while keeping it a valid prefix: an opening bracket is allowed whenever fewer than `n` have been used, and a closing bracket only when it would not outnumber the opens placed so far. Every complete path of length `2n` is a distinct well-formed string, and there are `Catalan(n)` of them. Time and space are O(4^n / sqrt(n)) for the output, with O(n) recursion depth.
