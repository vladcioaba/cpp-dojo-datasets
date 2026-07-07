## challenge: Longest Common Subsequence

tags: string, dynamic-programming
track: faang
difficulty: medium

Given two strings `text1` and `text2`, return the length of their longest common subsequence, or `0` if there is none. A subsequence is formed by deleting some (possibly zero) characters without reordering the rest; a common subsequence appears in both strings.

Constraints: `1 <= text1.length, text2.length <= 1000`, both consist of lowercase English letters. (Empty inputs yield `0`.)

Example: `text1 = "abcde", text2 = "ace"` → `3` (`"ace"`). Example: `text1 = "abc", text2 = "def"` → `0`.

hint: Compare the two strings position by position; matching characters extend a shared subsequence.
hint: Use a 2D DP where `dp[i][j]` is the LCS length of the first `i` characters of `text1` and the first `j` of `text2`.
hint: On a match take `dp[i-1][j-1] + 1`; otherwise take `max(dp[i-1][j], dp[i][j-1])`.

```cpp
// starter
#include <string>
int longestCommonSubsequence(std::string text1, std::string text2);
```

```cpp
int longestCommonSubsequence(std::string text1, std::string text2) {
    int m = (int)text1.size(), n = (int)text2.size();
    std::vector<std::vector<int>> dp(m + 1, std::vector<int>(n + 1, 0));
    for (int i = 1; i <= m; ++i)
        for (int j = 1; j <= n; ++j)
            dp[i][j] = (text1[i - 1] == text2[j - 1])
                         ? dp[i - 1][j - 1] + 1
                         : std::max(dp[i - 1][j], dp[i][j - 1]);
    return dp[m][n];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <algorithm>
using std::string;
//__USER__
int main() {
    if (longestCommonSubsequence("abcde", "ace") != 3) { std::puts("case1"); return 1; }
    if (longestCommonSubsequence("abc", "abc")   != 3) { std::puts("case2"); return 1; }
    if (longestCommonSubsequence("abc", "def")   != 0) { std::puts("case3"); return 1; }
    if (longestCommonSubsequence("", "abc")      != 0) { std::puts("case4"); return 1; }
    if (longestCommonSubsequence("abc", "")      != 0) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Fill a table where `dp[i][j]` is the LCS length of the prefixes `text1[0..i)` and `text2[0..j)`. Equal characters extend the diagonal subsequence by one; otherwise the best comes from dropping one character of either prefix. The bottom-right cell holds the answer, computed in O(m*n) time and O(m*n) space (reducible to O(min(m,n)) with rolling rows).
