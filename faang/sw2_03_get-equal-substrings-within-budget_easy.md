## challenge: Get Equal Substrings Within Budget
tags: string, sliding-window
track: faang
difficulty: easy

You are given two equal-length strings `s` and `t`. Changing the `i`-th character of `s` into the `i`-th character of `t` costs `|s[i] - t[i]|` (the absolute difference of their ASCII values). With a total budget of `maxCost`, return the length of the longest substring of `s` that can be transformed into the corresponding substring of `t` without exceeding the budget.

Constraints: `1 <= s.length <= 10^5`, `t.length == s.length`, `0 <= maxCost <= 10^6`, both strings consist of lowercase English letters.

Example: `s = "abcd", t = "bcdf", maxCost = 3` → `3` (converting `"abc"` to `"bcd"` costs `1+1+1 = 3`). Example: `s = "abcd", t = "cdef", maxCost = 3` → `1` (each conversion costs `2`). Example: `s = "abcd", t = "acde", maxCost = 0` → `1`.

hint: Think of a per-index cost array `|s[i] - t[i]|`; you want the longest contiguous run whose cost total stays within `maxCost`.

hint: Grow a window on the right, adding each index's cost; when the running cost exceeds the budget, advance the left edge and subtract costs until it fits again.

hint: Every position enters and leaves the window at most once, so a single linear scan suffices.

```cpp
// starter
#include <string>
int equalSubstring(std::string s, std::string t, int maxCost);
```

```cpp
int equalSubstring(std::string s, std::string t, int maxCost) {
    int left = 0, cost = 0, best = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        cost += std::abs(s[right] - t[right]);
        while (cost > maxCost) {
            cost -= std::abs(s[left] - t[left]);
            ++left;
        }
        best = std::max(best, right - left + 1);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <cstdlib>
#include <algorithm>
using std::string;
//__USER__
int main() {
    if (equalSubstring("abcd","bcdf",3)!=3) { std::puts("case1"); return 1; }
    if (equalSubstring("abcd","cdef",3)!=1) { std::puts("case2"); return 1; }
    if (equalSubstring("abcd","acde",0)!=1) { std::puts("case3"); return 1; }
    if (equalSubstring("krrgw","zjxss",19)!=2) { std::puts("case4"); return 1; }
    if (equalSubstring("abcd","abcd",100)!=4) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The transformation cost is additive and independent per index, so the task is the classic "longest window whose sum stays within a budget." Slide a variable-width window: extend the right edge and add the incoming index's conversion cost, and whenever the accumulated cost breaks the budget, retract the left edge, subtracting costs until the window is affordable again. Record the widest affordable window seen. Each index is added and removed once, giving O(n) time and O(1) space.
