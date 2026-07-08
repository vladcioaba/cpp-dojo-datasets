## challenge: Longest Palindromic Subsequence
tags: dynamic-programming, string
track: faang
difficulty: medium

Given a string `s`, return the length of the longest subsequence of `s` that reads the same forward and backward. A subsequence is formed by deleting zero or more characters without changing the order of the remaining characters.

Constraints: `1 <= s.length <= 1000`, `s` consists of lowercase English letters.

Example: `s = "bbbab"` → `4` (one longest palindromic subsequence is `"bbbb"`). Example: `s = "cbbd"` → `2` (the subsequence `"bb"`).

hint: Consider the substring between indices `i` and `j`; matching ends can wrap a palindrome from inside.
hint: If `s[i] == s[j]`, the answer is `dp[i+1][j-1] + 2`; otherwise it is `max(dp[i+1][j], dp[i][j-1])`.

```cpp
// starter
#include <string>
int longestPalindromeSubseq(std::string s);
```

```cpp
int longestPalindromeSubseq(std::string s) {
    int n = s.size();
    std::vector<std::vector<int>> dp(n, std::vector<int>(n, 0));
    for (int i = n - 1; i >= 0; --i) {
        dp[i][i] = 1;
        for (int j = i + 1; j < n; ++j) {
            if (s[i] == s[j]) dp[i][j] = dp[i+1][j-1] + 2;
            else dp[i][j] = std::max(dp[i+1][j], dp[i][j-1]);
        }
    }
    return dp[0][n-1];
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <algorithm>
//__USER__
int main() {
    if (longestPalindromeSubseq("bbbab")     != 4) { std::puts("case1"); return 1; }
    if (longestPalindromeSubseq("cbbd")      != 2) { std::puts("case2"); return 1; }
    if (longestPalindromeSubseq("a")         != 1) { std::puts("case3"); return 1; }
    if (longestPalindromeSubseq("racecar")   != 7) { std::puts("case4"); return 1; }
    if (longestPalindromeSubseq("abcdef")    != 1) { std::puts("case5"); return 1; }
    if (longestPalindromeSubseq("bananas")   != 5) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[i][j]` be the length of the longest palindromic subsequence of `s[i..j]`. A single character is a palindrome of length 1. When the boundary characters match they can wrap the best palindrome strictly inside, giving `dp[i+1][j-1] + 2`; otherwise we drop one end and take `max(dp[i+1][j], dp[i][j-1])`. Filling `i` from high to low ensures the inner subproblems are ready, and the answer is `dp[0][n-1]`. O(n^2) time and space.
