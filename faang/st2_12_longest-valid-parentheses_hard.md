## challenge: Longest Valid Parentheses
tags: stack, string, dynamic-programming
track: faang
difficulty: hard

Given a string `s` containing only the characters `(` and `)`, return the length of the longest contiguous substring that is a well-formed (valid) parentheses string.

Constraints: `0 <= s.length <= 3 * 10^4`; `s[i]` is either `(` or `)`.

Example: `s = "(()"` → `2` (the substring `"()"`). Example: `s = ")()())"` → `4` (the substring `"()()"`).

hint: Track positions, not just counts — you need the *length* of the matched region.
hint: Keep a stack of indices; seed it with a sentinel `-1` marking the boundary just before the current valid run.
hint: On `)`, pop; if the stack becomes empty, push this index as a new boundary, otherwise the run length is the current index minus the new top.

```cpp
// starter
#include <string>
int longestValidParentheses(std::string s);
```

```cpp
int longestValidParentheses(std::string s) {
    std::vector<int> st;
    st.push_back(-1);              // boundary before any valid substring
    int best = 0;
    for (int i = 0; i < (int)s.size(); ++i) {
        if (s[i] == '(') {
            st.push_back(i);
        } else {
            st.pop_back();
            if (st.empty()) st.push_back(i);          // new boundary
            else best = std::max(best, i - st.back());
        }
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <algorithm>
using std::string;
using std::vector;
//__USER__
int main() {
    if (longestValidParentheses("(()")      != 2) { std::puts("case1"); return 1; }
    if (longestValidParentheses(")()())")   != 4) { std::puts("case2"); return 1; }
    if (longestValidParentheses("")         != 0) { std::puts("case3"); return 1; }
    if (longestValidParentheses("()(())")   != 6) { std::puts("case4"); return 1; }
    if (longestValidParentheses("(()()")    != 4) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Use a stack of indices seeded with a sentinel `-1` that marks the position just before the current valid run. For each `(`, push its index. For each `)`, pop the top: if the stack becomes empty, this `)` cannot be matched, so push its index as the new left boundary; otherwise the top now holds the index right before the enclosing valid substring, and `i - st.back()` is the length of the valid run ending at `i`. Tracking the maximum of these lengths gives the answer in O(n) time and O(n) space.
