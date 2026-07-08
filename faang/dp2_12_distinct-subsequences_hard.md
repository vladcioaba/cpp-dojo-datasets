## challenge: Distinct Subsequences
tags: dynamic-programming, string
track: faang
difficulty: hard

Given two strings `s` and `t`, return the number of distinct subsequences of `s` that equal `t`. A subsequence is formed by deleting zero or more characters of `s` without reordering the rest.

Constraints: `0 <= s.length, t.length <= 1000`, both consist of English letters. The answer is guaranteed to fit in a 32-bit signed integer.

Example: `s = "rabbbit", t = "rabbit"` → `3` (the three ways to pick which `b` to drop). Example: `s = "babgbag", t = "bag"` → `5`.

hint: Match `t` against prefixes of `s`; at each `s` character you may use it or skip it.
hint: With a rolling 1-D table over `t`, update `j` from high to low so each `dp[j]` still reflects the previous `s` character.

```cpp
// starter
#include <string>
int numDistinct(std::string s, std::string t);
```

```cpp
int numDistinct(std::string s, std::string t) {
    int m = s.size(), n = t.size();
    std::vector<unsigned long long> dp(n + 1, 0);
    dp[0] = 1;
    for (int i = 1; i <= m; ++i)
        for (int j = n; j >= 1; --j)
            if (s[i-1] == t[j-1]) dp[j] += dp[j-1];
    return (int)dp[n];
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
//__USER__
int main() {
    if (numDistinct("rabbbit", "rabbit") != 3) { std::puts("case1"); return 1; }
    if (numDistinct("babgbag", "bag")    != 5) { std::puts("case2"); return 1; }
    if (numDistinct("abc", "")           != 1) { std::puts("case3"); return 1; }
    if (numDistinct("", "a")             != 0) { std::puts("case4"); return 1; }
    if (numDistinct("aaa", "aa")         != 3) { std::puts("case5"); return 1; }
    if (numDistinct("aaaaa", "a")        != 5) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[i][j]` be the number of subsequences of the first `i` characters of `s` that spell the first `j` characters of `t`. Every character of `s` may be skipped (`dp[i-1][j]`), and when `s[i-1] == t[j-1]` it may also be consumed (`+ dp[i-1][j-1]`). The empty target is matched exactly once, so `dp[i][0] = 1`. Collapsing to a single array over `j` and updating `j` from high to low keeps the needed "previous row" values intact; counts accumulate in unsigned 64-bit. O(m·n) time, O(n) space.
