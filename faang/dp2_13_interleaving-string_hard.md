## challenge: Interleaving String
tags: dynamic-programming, string
track: faang
difficulty: hard

Given strings `s1`, `s2`, and `s3`, return `true` if `s3` is formed by an interleaving of `s1` and `s2`. An interleaving keeps the internal order of each source string while merging their characters, so `|s1| + |s2|` must equal `|s3|`.

Constraints: `0 <= s1.length, s2.length <= 100`, `0 <= s3.length <= 200`, all strings consist of lowercase English letters.

Example: `s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"` → `true`. Example: `s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"` → `false`.

hint: After placing `i` characters of `s1` and `j` of `s2`, exactly `i+j` characters of `s3` are fixed.
hint: State `(i,j)` is reachable if the last placed character came from `s1` (`s1[i-1]==s3[i+j-1]`) with `(i-1,j)` reachable, or from `s2` similarly.

```cpp
// starter
#include <string>
bool isInterleave(std::string s1, std::string s2, std::string s3);
```

```cpp
bool isInterleave(std::string s1, std::string s2, std::string s3) {
    int m = s1.size(), n = s2.size();
    if (m + n != (int)s3.size()) return false;
    std::vector<char> dp(n + 1, 0);
    dp[0] = 1;
    for (int j = 1; j <= n; ++j)
        dp[j] = dp[j-1] && s2[j-1] == s3[j-1];
    for (int i = 1; i <= m; ++i) {
        dp[0] = dp[0] && s1[i-1] == s3[i-1];
        for (int j = 1; j <= n; ++j)
            dp[j] = (dp[j]   && s1[i-1] == s3[i-1+j]) ||
                    (dp[j-1] && s2[j-1] == s3[i-1+j]);
    }
    return dp[n];
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
//__USER__
int main() {
    if (isInterleave("aabcc", "dbbca", "aadbbcbcac") != true)  { std::puts("case1"); return 1; }
    if (isInterleave("aabcc", "dbbca", "aadbbbaccc") != false) { std::puts("case2"); return 1; }
    if (isInterleave("", "", "")                     != true)  { std::puts("case3"); return 1; }
    if (isInterleave("", "abc", "abc")               != true)  { std::puts("case4"); return 1; }
    if (isInterleave("a", "b", "ab")                 != true)  { std::puts("case5"); return 1; }
    if (isInterleave("a", "b", "aa")                 != false) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[i][j]` be true when the first `i` characters of `s1` and the first `j` of `s2` interleave to form the first `i+j` characters of `s3`. The final character `s3[i+j-1]` came either from `s1` (needs `dp[i-1][j]` and `s1[i-1]==s3[i+j-1]`) or from `s2` (needs `dp[i][j-1]` and `s2[j-1]==s3[i+j-1]`). Lengths must match or the answer is immediately false. A single row over `j`, refreshed per `i`, holds both `dp[i-1][j]` (its old value) and `dp[i][j-1]` (already updated). O(m·n) time, O(n) space.
