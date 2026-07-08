## challenge: Palindromic Substrings
tags: dynamic-programming, string, two-pointers
track: faang
difficulty: medium

Given a string `s`, return the number of palindromic substrings in it. A substring is a contiguous sequence of characters, and two substrings are counted separately if they start or end at different positions, even if they consist of the same letters.

Constraints: `1 <= s.length <= 1000`, `s` consists of lowercase English letters.

Example: `s = "abc"` → `3` (the substrings `"a"`, `"b"`, `"c"`). Example: `s = "aaa"` → `6` (`"a"` x3, `"aa"` x2, `"aaa"` x1).

hint: Every palindrome has a center — either a single character or the gap between two characters.
hint: Expand outward from each of the `2n-1` centers while the mirrored characters keep matching.

```cpp
// starter
#include <string>
int countSubstrings(std::string s);
```

```cpp
int countSubstrings(std::string s) {
    int n = s.size(), count = 0;
    for (int c = 0; c < 2 * n - 1; ++c) {
        int l = c / 2, r = l + (c & 1);
        while (l >= 0 && r < n && s[l] == s[r]) {
            ++count;
            --l;
            ++r;
        }
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
//__USER__
int main() {
    if (countSubstrings("abc")     != 3)  { std::puts("case1"); return 1; }
    if (countSubstrings("aaa")     != 6)  { std::puts("case2"); return 1; }
    if (countSubstrings("a")       != 1)  { std::puts("case3"); return 1; }
    if (countSubstrings("aba")     != 4)  { std::puts("case4"); return 1; }
    if (countSubstrings("abccba")  != 9)  { std::puts("case5"); return 1; }
    if (countSubstrings("aaaaa")   != 15) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A palindrome is determined by its center, and a string of length `n` has `2n-1` possible centers — `n` on single characters and `n-1` between adjacent characters. From each center, extend a pair of pointers outward as long as they stay in bounds and the characters match; every successful match is one more palindromic substring. This center-expansion runs in O(n^2) time with O(1) extra space, matching the classic interval-DP count without the table.
