## challenge: Evaluate Reverse Polish Notation

tags: stack, math, array
track: faang
difficulty: medium

You are given an array of strings `tokens` representing an arithmetic expression in Reverse Polish Notation. Evaluate the expression and return the resulting integer. Valid operators are `+`, `-`, `*`, and `/`; each operand is an integer, division truncates toward zero, and the input is always a well-formed expression.

Constraints: `1 <= tokens.length <= 10^4`; each token is `+`, `-`, `*`, `/`, or an integer in `[-200, 200]`; the answer and all intermediate results fit in a 32-bit signed integer.

Example: `tokens = ["2","1","+","3","*"]` → `9` (`(2+1)*3`). Example: `tokens = ["4","13","5","/","+"]` → `6` (`4 + 13/5`).

hint: Postfix notation removes the need for parentheses — operands come before the operator that consumes them.
hint: Scan left to right with a stack: push numbers, and on an operator pop the two most recent operands.
hint: Order matters for `-` and `/`: the operand popped second is the left-hand side, so compute `a op b` where `b` was popped first.

```cpp
// starter
#include <vector>
#include <string>
int evalRPN(std::vector<std::string>& tokens);
```

```cpp
int evalRPN(std::vector<std::string>& tokens) {
    std::vector<int> st;
    for (const std::string& t : tokens) {
        if (t == "+" || t == "-" || t == "*" || t == "/") {
            int b = st.back(); st.pop_back();
            int a = st.back(); st.pop_back();
            int r = 0;
            if (t == "+") r = a + b;
            else if (t == "-") r = a - b;
            else if (t == "*") r = a * b;
            else r = a / b;  // C++ integer division truncates toward zero
            st.push_back(r);
        } else {
            st.push_back(std::stoi(t));
        }
    }
    return st.back();
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
int main() {
    { vector<string> t{"2","1","+","3","*"};       if (evalRPN(t) != 9)   { std::puts("case1"); return 1; } }
    { vector<string> t{"4","13","5","/","+"};       if (evalRPN(t) != 6)   { std::puts("case2"); return 1; } }
    { vector<string> t{"10","6","9","3","+","-11","*","/","*","17","+","5","+"};
                                                    if (evalRPN(t) != 22)  { std::puts("case3"); return 1; } }
    { vector<string> t{"3","-4","*"};               if (evalRPN(t) != -12) { std::puts("case4"); return 1; } }
    { vector<string> t{"-42"};                       if (evalRPN(t) != -42) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Process tokens left to right using an operand stack. Numbers are pushed; each operator pops its two operands, applies the operation (the earlier-pushed value is the left operand), and pushes the result. C++ integer division already truncates toward zero, matching the required semantics. The single remaining stack value is the answer, computed in O(n) time and O(n) space.
