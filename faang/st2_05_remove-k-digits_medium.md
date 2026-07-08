## challenge: Remove K Digits
tags: monotonic-stack, stack, greedy, string
track: faang
difficulty: medium

Given a non-negative integer represented as a string `num` and an integer `k`, remove exactly `k` digits so that the resulting number is the smallest possible. Return that number as a string with no leading zeros; if the result is empty, return `"0"`.

Constraints: `1 <= k <= num.length <= 10^5`; `num` consists of digits only and has no leading zeros except when it is `"0"` itself.

Example: `num = "1432219", k = 3` → `"1219"`. Example: `num = "10200", k = 1` → `"200"` (remove the leading `1`).

hint: A digit hurts the value most when a larger digit sits to the left of a smaller one — remove the left, larger digit first.
hint: Sweep left to right and, while you still have removals left and the last kept digit exceeds the current one, pop it.
hint: If removals remain after the sweep, drop from the end; finally strip leading zeros.

```cpp
// starter
#include <string>
std::string removeKdigits(std::string num, int k);
```

```cpp
std::string removeKdigits(std::string num, int k) {
    std::string st;
    for (char c : num) {
        while (k > 0 && !st.empty() && st.back() > c) {
            st.pop_back();
            --k;
        }
        st.push_back(c);
    }
    while (k > 0) { st.pop_back(); --k; }   // still need to remove: drop largest suffix digits
    int i = 0;
    while (i < (int)st.size() && st[i] == '0') ++i;   // strip leading zeros
    std::string res = st.substr(i);
    return res.empty() ? "0" : res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (removeKdigits("1432219", 3) != "1219") { std::puts("case1"); return 1; }
    if (removeKdigits("10200", 1)   != "200")  { std::puts("case2"); return 1; }
    if (removeKdigits("10", 2)      != "0")    { std::puts("case3"); return 1; }
    if (removeKdigits("112", 1)     != "11")   { std::puts("case4"); return 1; }
    if (removeKdigits("1234567890", 9) != "0") { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** To minimize the number, greedily keep a digit sequence that is as non-decreasing as possible. Using a stack, for each incoming digit pop any strictly larger digit still on top while removals remain — this deletes an expensive high digit that sits before a smaller one. If budget `k` is left over after the pass (the input was non-decreasing), remove from the tail. Finally strip leading zeros and guard against an empty result. Each digit is pushed and popped at most once, so the algorithm runs in O(n) time and O(n) space.
