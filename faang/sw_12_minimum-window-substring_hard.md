## challenge: Minimum Window Substring
tags: string, hash-table, sliding-window
track: faang
difficulty: hard

Given strings `s` and `t`, return the shortest contiguous substring of `s` that contains every character of `t`, including duplicates. If no such window exists, return the empty string `""`. The answer is guaranteed to be unique whenever it exists.

Constraints: `1 <= s.length, t.length <= 10^5`, both strings consist of uppercase and lowercase English letters.

Example: `s = "ADOBECODEBANC", t = "ABC"` → `"BANC"`. Example: `s = "a", t = "a"` → `"a"`. Example: `s = "a", t = "aa"` → `""` (only one `a` is available).

hint: Count how many of each character `t` requires, and track how many of those requirements the current window still has left to satisfy.
hint: Expand the right edge; each time a character reaches the needed multiplicity, one fewer requirement remains.
hint: Once every requirement is met, shrink from the left to find the tightest valid window and record it before a requirement breaks again.

```cpp
// starter
#include <string>
std::string minWindow(std::string s, std::string t);
```

```cpp
std::string minWindow(std::string s, std::string t) {
    std::array<int,128> need{};
    for (char c : t) ++need[(unsigned char)c];
    int required = t.size();
    int left = 0, bestLen = INT_MAX, bestStart = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        if (need[(unsigned char)s[right]]-- > 0) --required;
        while (required == 0) {
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }
            if (++need[(unsigned char)s[left]] > 0) ++required;
            ++left;
        }
    }
    return bestLen == INT_MAX ? std::string() : s.substr(bestStart, bestLen);
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <array>
#include <climits>
using std::string;
//__USER__
int main() {
    if (minWindow("ADOBECODEBANC","ABC") != "BANC") { std::puts("case1"); return 1; }
    if (minWindow("a","a") != "a") { std::puts("case2"); return 1; }
    if (minWindow("a","aa") != "") { std::puts("case3"); return 1; }
    if (minWindow("aa","aa") != "aa") { std::puts("case4"); return 1; }
    if (minWindow("cabwefgewcwaefgcf","cae") != "cwae") { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Track how many required characters remain unsatisfied with a single counter `required`, initialized to `t.length`. As the right edge expands, decrement each character's need; when a needed character's count crosses from positive to zero-or-below, one requirement is satisfied and `required` drops. While `required == 0` the window is valid, so shrink from the left, recording the smallest window, and stop shrinking the moment removing a character breaks a requirement. Using a 128-slot ASCII table makes every update O(1); overall O(|s| + |t|) time and O(1) space.
