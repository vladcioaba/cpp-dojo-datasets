## challenge: Edit Distance
tags: dynamic-programming, string
track: faang
difficulty: hard

Given two strings `word1` and `word2`, return the minimum number of single-character operations required to transform `word1` into `word2`. The allowed operations are inserting a character, deleting a character, and replacing a character.

Constraints: `0 <= word1.length, word2.length <= 500`, both consist of lowercase English letters.

Example: `word1 = "horse", word2 = "ros"` → `3` (`horse -> rorse -> rose -> ros`). Example: `word1 = "intention", word2 = "execution"` → `5`.

hint: Compare the two strings prefix by prefix; the last characters either already match or need one edit.
hint: Let `dp[i][j]` be the edit distance between the first `i` characters of `word1` and the first `j` of `word2`.
hint: If the characters match, `dp[i][j] = dp[i-1][j-1]`; otherwise it is `1 + min(replace, delete, insert)` = `1 + min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])`.

```cpp
// starter
#include <string>
int minDistance(std::string word1, std::string word2);
```

```cpp
int minDistance(std::string word1, std::string word2) {
    int m = (int)word1.size(), n = (int)word2.size();
    std::vector<int> dp(n + 1);
    for (int j = 0; j <= n; ++j) dp[j] = j;
    for (int i = 1; i <= m; ++i) {
        int prev = dp[0];  // dp[i-1][j-1]
        dp[0] = i;
        for (int j = 1; j <= n; ++j) {
            int tmp = dp[j];
            if (word1[i - 1] == word2[j - 1]) dp[j] = prev;
            else dp[j] = 1 + std::min({prev, dp[j], dp[j - 1]});
            prev = tmp;
        }
    }
    return dp[n];
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <algorithm>
using std::string;
using std::vector;
//__USER__
int main() {
    if (minDistance("horse", "ros")           != 3) { std::puts("case1"); return 1; }
    if (minDistance("intention", "execution") != 5) { std::puts("case2"); return 1; }
    if (minDistance("", "abc")                != 3) { std::puts("case3"); return 1; }
    if (minDistance("abc", "")                != 3) { std::puts("case4"); return 1; }
    if (minDistance("abc", "abc")             != 0) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** dp[i][j] is the edit distance between the first i characters of word1 and the first j of word2, seeded with dp[i][0] = i and dp[0][j] = j. Matching last characters cost nothing (dp[i-1][j-1]); otherwise take one plus the cheapest of replace, delete, or insert. Rolling a single row with a saved diagonal gives O(m*n) time and O(n) space.
