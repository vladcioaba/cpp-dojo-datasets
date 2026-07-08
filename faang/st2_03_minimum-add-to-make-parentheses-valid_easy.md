## challenge: Minimum Add to Make Parentheses Valid
tags: stack, string, greedy
track: faang
difficulty: easy

A parentheses string is valid if every `(` has a matching `)` and pairs are properly nested. Given a string `s` of only `(` and `)`, in one move you may insert a single `(` or `)` at any position. Return the minimum number of insertions needed to make `s` valid.

Constraints: `1 <= s.length <= 1000`; `s` consists only of the characters `(` and `)`.

Example: `s = "())"` → `1` (insert one `(`). Example: `s = "((("` → `3` (insert three `)`).

hint: A `)` is only satisfiable if there is an unmatched `(` waiting for it.
hint: Track the number of currently unmatched `(`; when a `)` finds none, it needs a fresh insertion.
hint: The answer is the count of unmatchable `)` plus the `(` still unmatched at the end.

```cpp
// starter
#include <string>
int minAddToMakeValid(std::string s);
```

```cpp
int minAddToMakeValid(std::string s) {
    int open = 0;   // unmatched '(' so far (stack size)
    int add = 0;    // insertions forced by unmatched ')'
    for (char c : s) {
        if (c == '(') ++open;
        else if (open > 0) --open;
        else ++add;
    }
    return add + open;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (minAddToMakeValid("())")    != 1) { std::puts("case1"); return 1; }
    if (minAddToMakeValid("(((")    != 3) { std::puts("case2"); return 1; }
    if (minAddToMakeValid("()")     != 0) { std::puts("case3"); return 1; }
    if (minAddToMakeValid("()))((") != 4) { std::puts("case4"); return 1; }
    if (minAddToMakeValid(")))")    != 3) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Scan left to right keeping `open`, the number of `(` still awaiting a match — effectively the height of a parenthesis stack. Each `(` increments it; each `)` either matches an open one (decrement) or, finding none, forces an insertion of a `(` (`add`). After the scan, the remaining `open` unmatched `(` each need a `)` appended. The total `add + open` is the minimum number of insertions, computed in O(n) time and O(1) space.
