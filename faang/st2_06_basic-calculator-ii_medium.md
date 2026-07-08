## challenge: Basic Calculator II
tags: stack, math, string
track: faang
difficulty: medium

Given a string `s` representing a valid arithmetic expression, evaluate it and return the result as an integer. The expression contains non-negative integers and the operators `+`, `-`, `*`, `/`, separated by optional spaces. Multiplication and division have higher precedence than addition and subtraction, and integer division truncates toward zero.

Constraints: `1 <= s.length <= 3 * 10^5`; `s` consists of integers and the operators `+`, `-`, `*`, `/` separated by spaces; `s` is a valid expression; all intermediate values fit in a 32-bit signed integer.

Example: `s = "3+2*2"` → `7` (`3 + 4`). Example: `s = " 3/2 "` → `1`. Example: `s = " 3+5 / 2 "` → `5`.

hint: Precedence with only `+ - * /` can be handled in one pass without a full parser.
hint: Push each term onto a stack, but apply `*` and `/` immediately to the top since they bind tighter.
hint: Remember the pending operator; parse a full number, then combine it with the stack when the next operator (or the end) arrives. The answer is the sum of the stack.

```cpp
// starter
#include <string>
int calculate(std::string s);
```

```cpp
int calculate(std::string s) {
    std::vector<int> st;
    long num = 0;
    char op = '+';
    int n = (int)s.size();
    for (int i = 0; i < n; ++i) {
        char c = s[i];
        if (std::isdigit((unsigned char)c)) num = num * 10 + (c - '0');
        if ((!std::isdigit((unsigned char)c) && c != ' ') || i == n - 1) {
            if (op == '+') st.push_back((int)num);
            else if (op == '-') st.push_back((int)-num);
            else if (op == '*') { int t = st.back(); st.pop_back(); st.push_back((int)(t * num)); }
            else { int t = st.back(); st.pop_back(); st.push_back((int)(t / num)); }
            op = c;
            num = 0;
        }
    }
    int sum = 0;
    for (int x : st) sum += x;
    return sum;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <cctype>
using std::string;
using std::vector;
//__USER__
int main() {
    if (calculate("3+2*2")     != 7)  { std::puts("case1"); return 1; }
    if (calculate(" 3/2 ")     != 1)  { std::puts("case2"); return 1; }
    if (calculate(" 3+5 / 2 ") != 5)  { std::puts("case3"); return 1; }
    if (calculate("14-3/2")    != 13) { std::puts("case4"); return 1; }
    if (calculate("100000000") != 100000000) { std::puts("case5"); return 1; }
    if (calculate("2*3+4*5")   != 26) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Scan the string accumulating each multi-digit number, and remember the operator that precedes it (`op`, initially `+`). When a new operator or the end of the string is reached, resolve the pending number: `+` and `-` push the value (negated for `-`), deferring their evaluation, while `*` and `/` immediately combine with the stack's top because they bind tighter. Summing the stack at the end applies the low-precedence additions. This handles operator precedence in a single O(n) pass with O(n) space.
