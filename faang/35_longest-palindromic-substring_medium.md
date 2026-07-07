## challenge: Longest Palindromic Substring

tags: string, dynamic-programming, two-pointers
track: faang
difficulty: medium

Given a string `s`, return the longest contiguous substring of `s` that reads the same forward and backward. If several substrings share the maximum length, returning any one of them is acceptable.

Constraints: `1 <= s.length <= 1000`, `s` consists of digits and English letters. (The empty string maps to the empty answer.)

Example: `s = "babad"` → `"bab"` (or `"aba"`), length `3`. Example: `s = "cbbd"` → `"bb"`, length `2`.

hint: Every palindrome is defined by a center; there are `2n-1` possible centers (each character and each gap between characters).
hint: Expand around each center, growing left and right while the characters match, and track the widest expansion seen.
hint: Handle both odd-length centers (a single character) and even-length centers (a pair) to catch palindromes like `"bb"`.

```cpp
// starter
#include <string>
std::string longestPalindrome(std::string s);
```

```cpp
std::string longestPalindrome(std::string s) {
    if (s.empty()) return "";
    int start = 0, best = 1;
    auto expand = [&](int l, int r) {
        while (l >= 0 && r < (int)s.size() && s[l] == s[r]) { --l; ++r; }
        int len = r - l - 1;  // palindrome is s[l+1 .. r-1]
        if (len > best) { best = len; start = l + 1; }
    };
    for (int i = 0; i < (int)s.size(); ++i) {
        expand(i, i);      // odd-length center
        expand(i, i + 1);  // even-length center
    }
    return s.substr(start, best);
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
static bool isPalin(const string& t) {
    int i = 0, j = (int)t.size() - 1;
    while (i < j) { if (t[i] != t[j]) return false; ++i; --j; }
    return true;
}
// The answer need not be unique, so validate shape rather than an exact string:
// (a) substring of s, (b) a palindrome, (c) length equals the known maximum.
static bool ok(const string& s, const string& ans, int expectedLen) {
    if ((int)ans.size() != expectedLen) return false;
    if (!isPalin(ans)) return false;
    if (s.find(ans) == string::npos) return false;
    return true;
}
//__USER__
int main() {
    if (!ok("babad", longestPalindrome("babad"), 3))                       { std::puts("case1"); return 1; }
    if (!ok("cbbd", longestPalindrome("cbbd"), 2))                         { std::puts("case2"); return 1; }
    if (!ok("a", longestPalindrome("a"), 1))                               { std::puts("case3"); return 1; }
    if (!ok("", longestPalindrome(""), 0))                                 { std::puts("case4"); return 1; }
    if (!ok("forgeeksskeegfor", longestPalindrome("forgeeksskeegfor"), 10)){ std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Treat each of the `2n-1` centers as the middle of a candidate palindrome and expand outward while the mirrored characters match, remembering the longest span. Considering both single-character and between-character centers covers odd- and even-length palindromes. This runs in O(n^2) time and O(1) extra space, simpler than the O(n^2) DP table and sufficient for `n <= 1000`.
