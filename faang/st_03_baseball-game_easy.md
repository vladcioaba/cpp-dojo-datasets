## challenge: Baseball Game
tags: stack, array, simulation
track: faang
difficulty: easy

You are keeping score for a game with a special record. You are given a list of strings `operations`, applied in order. Each operation is one of: an integer `x` (record a new score of `x`); `"+"` (record a new score equal to the sum of the previous two scores); `"D"` (record a new score that is double the previous score); or `"C"` (invalidate and remove the previous score). Every `"+"`, `"D"`, and `"C"` refers to scores that exist when it is applied. Return the sum of all scores remaining on the record after all operations.

Constraints: `1 <= operations.length <= 1000`; each `operations[i]` is `"C"`, `"D"`, `"+"`, or a string representing an integer in `[-3 * 10^4, 3 * 10^4]`; the final sum fits in a 32-bit signed integer.

Example: `["5","2","C","D","+"]` → `30`. Example: `["5","-2","4","C","D","9","+","+"]` → `27`. Example: `["1"]` → `1`.

hint: The record behaves like a stack: new scores go on top, and `"C"` removes the top.
hint: `"+"` needs the top two scores and `"D"` needs the top one — both are cheap on a stack.
hint: After processing every operation, the answer is simply the sum of everything left on the stack.

```cpp
// starter
#include <vector>
#include <string>
int calPoints(std::vector<std::string>& operations);
```

```cpp
int calPoints(std::vector<std::string>& operations) {
    std::vector<int> st;
    for (const std::string& op : operations) {
        if (op == "+") {
            int a = st.back();
            int b = st[st.size() - 2];
            st.push_back(a + b);
        } else if (op == "D") {
            st.push_back(2 * st.back());
        } else if (op == "C") {
            st.pop_back();
        } else {
            st.push_back(std::stoi(op));
        }
    }
    int sum = 0;
    for (int v : st) sum += v;
    return sum;
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
    { vector<string> o{"5","2","C","D","+"};                     if (calPoints(o) != 30) { std::puts("case1"); return 1; } }
    { vector<string> o{"5","-2","4","C","D","9","+","+"};        if (calPoints(o) != 27) { std::puts("case2"); return 1; } }
    { vector<string> o{"1"};                                     if (calPoints(o) != 1)  { std::puts("case3"); return 1; } }
    { vector<string> o{"1","C"};                                 if (calPoints(o) != 0)  { std::puts("case4"); return 1; } }
    { vector<string> o{"-5","-10","+"};                          if (calPoints(o) != -30) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Treat the record as a stack of integers. Push parsed integers directly; for `"D"` push twice the current top; for `"+"` push the sum of the top two; for `"C"` pop the top. Because each operation only touches the top one or two entries, the whole pass is O(n) time, after which summing the stack yields the score. O(n) space.
