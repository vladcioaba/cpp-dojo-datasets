## challenge: Generate Parentheses
tags: stack, backtracking, string
track: faang
difficulty: medium

Given `n` pairs of parentheses, generate all combinations of well-formed parentheses. A string is well-formed if every opening bracket has a matching closing bracket that comes after it and no prefix has more closing than opening brackets. Return all such strings; the order of the returned list does not matter.

Constraints: `1 <= n <= 8`.

Example: `n = 3` → `["((()))","(()())","(())()","()(())","()()()"]`. Example: `n = 1` → `["()"]`.

hint: Build strings character by character; at each step you either open a new bracket or close an existing one.
hint: A partial string is valid to extend as long as opens used `<= n` and closes used `< opens` (the balance, like a stack of unmatched `(`, never goes negative).
hint: Recurse (backtrack): add `(` when you still have opens left, add `)` when there is an unmatched `(` to close, and record the string when its length reaches `2n`.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> generateParenthesis(int n);
```

```cpp
static void build(int open, int close, int n, std::string& cur, std::vector<std::string>& res) {
    if ((int)cur.size() == 2 * n) { res.push_back(cur); return; }
    if (open < n) {                      // open a new bracket
        cur.push_back('(');
        build(open + 1, close, n, cur, res);
        cur.pop_back();
    }
    if (close < open) {                  // close an unmatched bracket
        cur.push_back(')');
        build(open, close + 1, n, cur, res);
        cur.pop_back();
    }
}
std::vector<std::string> generateParenthesis(int n) {
    std::vector<std::string> res;
    std::string cur;
    build(0, 0, n, cur, res);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <algorithm>
using std::vector;
using std::string;
//__USER__
int main() {
    { auto r = generateParenthesis(1);
      if (r != vector<string>({"()"})) { std::puts("case1"); return 1; } }
    { auto r = generateParenthesis(2); std::sort(r.begin(), r.end());
      if (r != vector<string>({"(())","()()"})) { std::puts("case2"); return 1; } }
    { auto r = generateParenthesis(3); std::sort(r.begin(), r.end());
      vector<string> exp{"((()))","(()())","(())()","()(())","()()()"};
      std::sort(exp.begin(), exp.end());
      if (r != exp) { std::puts("case3"); return 1; } }
    { auto r = generateParenthesis(4);
      if (r.size() != 14) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Backtrack over the string being built, tracking how many `(` and `)` have been placed. You may add `(` while fewer than `n` opens are used, and you may add `)` only while there are more opens than closes (equivalently, while the running stack of unmatched `(` is non-empty). When the string reaches length `2n` it is complete and valid, so record it. The number of results is the n-th Catalan number, and the search visits each partial exactly once.
