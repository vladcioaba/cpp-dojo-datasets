## challenge: Valid Parentheses
tags: stack, string
track: faang
difficulty: easy

Given a string `s` containing only the characters `()[]{}`, determine if the brackets are validly matched: every opening bracket is closed by the same type, and brackets close in the correct order.

Constraints: `1 <= s.length <= 10^4`.

Example: `"()[]{}"` → `true`. Example: `"(]"` → `false`. Example: `"([)]"` → `false`. Example: `"{[]}"` → `true`.

```cpp
// starter
#include <string>
bool isValid(std::string s);
```

```cpp
bool isValid(std::string s) {
    std::vector<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            st.push_back(c);
        } else {
            if (st.empty()) return false;
            char t = st.back();
            st.pop_back();
            if ((c == ')' && t != '(') ||
                (c == ']' && t != '[') ||
                (c == '}' && t != '{')) return false;
        }
    }
    return st.empty();
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
//__USER__
int main() {
    if (!isValid("()"))       { std::puts("case1"); return 1; }
    if (!isValid("()[]{}"))   { std::puts("case2"); return 1; }
    if ( isValid("(]"))       { std::puts("case3"); return 1; }
    if ( isValid("([)]"))     { std::puts("case4"); return 1; }
    if (!isValid("{[]}"))     { std::puts("case5"); return 1; }
    if ( isValid("("))        { std::puts("case6"); return 1; }
    if ( isValid("]"))        { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```
