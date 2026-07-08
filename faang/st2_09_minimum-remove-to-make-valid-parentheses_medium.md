## challenge: Minimum Remove to Make Valid Parentheses
tags: stack, string
track: faang
difficulty: medium

Given a string `s` of lowercase letters and the characters `(` and `)`, remove the minimum number of parentheses (`(` or `)`, in any positions) so that the resulting string is valid and is as long as possible. A string is valid if it is empty, contains only lowercase letters, or can be written as `AB` or `(A)` where `A` and `B` are valid. Return any valid result of minimum removals.

Constraints: `1 <= s.length <= 10^5`; `s[i]` is either a lowercase English letter, `(`, or `)`.

Example: `s = "lee(t(c)o)de)"` → `"lee(t(c)o)de"` (remove the last `)`). Example: `s = "a)b(c)d"` → `"ab(c)d"`.

hint: A `)` is invalid when there is no unmatched `(` before it; an `(` is invalid if it is never closed.
hint: Push the indices of `(` onto a stack; on `)`, either match by popping or mark this `)` for removal.
hint: Any `(` indices still on the stack at the end are unmatched — mark them too, then build the string skipping all marked positions.

```cpp
// starter
#include <string>
std::string minRemoveToMakeValid(std::string s);
```

```cpp
std::string minRemoveToMakeValid(std::string s) {
    std::vector<int> stk;                    // indices of unmatched '('
    std::vector<bool> drop(s.size(), false);
    for (int i = 0; i < (int)s.size(); ++i) {
        if (s[i] == '(') stk.push_back(i);
        else if (s[i] == ')') {
            if (!stk.empty()) stk.pop_back();
            else drop[i] = true;             // unmatched ')'
        }
    }
    for (int idx : stk) drop[idx] = true;    // leftover unmatched '('
    std::string res;
    for (int i = 0; i < (int)s.size(); ++i)
        if (!drop[i]) res.push_back(s[i]);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
using std::string;
using std::vector;
//__USER__
int main() {
    if (minRemoveToMakeValid("lee(t(c)o)de)") != "lee(t(c)o)de") { std::puts("case1"); return 1; }
    if (minRemoveToMakeValid("a)b(c)d")       != "ab(c)d")       { std::puts("case2"); return 1; }
    if (minRemoveToMakeValid("))((")          != "")            { std::puts("case3"); return 1; }
    if (minRemoveToMakeValid("(a(b(c)d)")     != "a(b(c)d)")     { std::puts("case4"); return 1; }
    if (minRemoveToMakeValid("abc")           != "abc")          { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Make one pass tracking the indices of unmatched `(` on a stack. A `)` either matches the most recent `(` (pop it) or, finding none, is itself unmatched and marked for removal. After the pass, any `(` indices remaining on the stack were never closed, so mark them too. Building the output while skipping all marked indices yields a valid string, and because only the provably unmatchable parentheses are removed, the count is minimal. The whole procedure is O(n) time and O(n) space.
