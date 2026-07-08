## challenge: Expression Add Operators
tags: backtracking, string, math
track: faang
difficulty: hard

Given a string `num` that contains only digits and an integer `target`, return all expressions you can build by inserting the binary operators `'+'`, `'-'`, and/or `'*'` between the digits of `num` (keeping the digits in their given order) so that the resulting expression evaluates to `target`. Standard precedence applies: `'*'` binds tighter than `'+'` and `'-'`. An operand may not carry a redundant leading zero (so `"05"` is not allowed, though a single `"0"` is). Return the expressions in any order.

Constraints: `1 <= num.length <= 10`, `num` consists only of digits, `-2^31 <= target <= 2^31 - 1`.

Example: `num = "123", target = 6` -> `["1*2*3","1+2+3"]`. Example: `num = "232", target = 8` -> `["2*3+2","2+3*2"]`. Example: `num = "105", target = 5` -> `["1*0+5","10-5"]`.

hint: Scan left to right choosing the next operand; between operands try each of `+`, `-`, `*`.
hint: Multiplication disrupts left-to-right evaluation, so carry the value of the last operand so you can undo and reapply it: `cur - prev + prev * val`.
hint: Prevent leading-zero operands by stopping the operand-extension loop as soon as the operand would start with `'0'`.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> addOperators(std::string num, int target);
```

```cpp
std::vector<std::string> addOperators(std::string num, int target) {
    std::vector<std::string> res;
    int n = (int)num.size();
    std::function<void(int, long long, long long, std::string)> dfs =
        [&](int pos, long long cur, long long prev, std::string expr) {
        if (pos == n) {
            if (cur == (long long)target) res.push_back(expr);
            return;
        }
        for (int i = pos; i < n; ++i) {
            if (i > pos && num[pos] == '0') break;
            std::string part = num.substr(pos, i - pos + 1);
            long long val = std::stoll(part);
            if (pos == 0) {
                dfs(i + 1, val, val, part);
            } else {
                dfs(i + 1, cur + val, val, expr + "+" + part);
                dfs(i + 1, cur - val, -val, expr + "-" + part);
                dfs(i + 1, cur - prev + prev * val, prev * val, expr + "*" + part);
            }
        }
    };
    dfs(0, 0, 0, "");
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
//__USER__
int main() {
    { auto want = canon({"1*2*3","1+2+3"});
      if (canon(addOperators("123", 6)) != want) { std::puts("case1"); return 1; } }
    { auto want = canon({"2*3+2","2+3*2"});
      if (canon(addOperators("232", 8)) != want) { std::puts("case2"); return 1; } }
    { auto want = canon({"1*0+5","10-5"});
      if (canon(addOperators("105", 5)) != want) { std::puts("case3"); return 1; } }
    { auto want = canon({"0*0","0+0","0-0"});
      if (canon(addOperators("00", 0)) != want) { std::puts("case4"); return 1; } }
    { if (!addOperators("3456237490", 9191).empty()) { std::puts("case5"); return 1; } }
    { auto want = canon({"1"});
      if (canon(addOperators("1", 1)) != want) { std::puts("case6"); return 1; } }
    { auto want = canon({"12"});
      if (canon(addOperators("12", 12)) != want) { std::puts("case7"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Backtrack over the string, at each step consuming the next operand and, except at the very start, trying `+`, `-`, and `*` before it. The wrinkle is precedence: because `*` must combine with the immediately preceding operand rather than the whole running total, the recursion carries `prev` (the last operand's signed contribution) and evaluates a multiply as `cur - prev + prev * val`, updating `prev` to `prev * val`. Stopping the operand loop when it would begin with `'0'` enforces the no-leading-zero rule. There are O(4^n) ways to split-and-operate, each costing O(n) to build the string.
