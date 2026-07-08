## challenge: Word Break
tags: dynamic-programming, string
track: faang
difficulty: medium

Given a string `s` and a dictionary `wordDict` of strings, return `true` if `s` can be split into a sequence of one or more dictionary words separated by no extra characters. Dictionary words may be reused any number of times.

Constraints: `1 <= s.length <= 300`, `1 <= wordDict.length <= 1000`, `1 <= wordDict[i].length <= 20`. All strings consist of lowercase English letters.

Example: `s = "leetcode", wordDict = ["leet","code"]` → `true`. Example: `s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]` → `false`.

hint: A prefix of `s` is breakable if it ends with a dictionary word whose start is itself a breakable prefix.
hint: Let `dp[i]` mean "the first `i` characters of `s` can be segmented"; `dp[0]` is true.
hint: `dp[i]` is true if some `j < i` has `dp[j]` true and `s[j..i)` is in the dictionary.

```cpp
// starter
#include <string>
#include <vector>
bool wordBreak(std::string s, std::vector<std::string>& wordDict);
```

```cpp
bool wordBreak(std::string s, std::vector<std::string>& wordDict) {
    std::unordered_set<std::string> dict(wordDict.begin(), wordDict.end());
    int n = (int)s.size();
    std::vector<char> dp(n + 1, false);
    dp[0] = true;
    for (int i = 1; i <= n; ++i)
        for (int j = 0; j < i; ++j)
            if (dp[j] && dict.count(s.substr(j, i - j))) { dp[i] = true; break; }
    return dp[n];
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <unordered_set>
using std::vector;
using std::string;
//__USER__
int main() {
    { vector<string> d{"leet","code"}; if (!wordBreak("leetcode", d)) { std::puts("case1"); return 1; } }
    { vector<string> d{"apple","pen"}; if (!wordBreak("applepenapple", d)) { std::puts("case2"); return 1; } }
    { vector<string> d{"cats","dog","sand","and","cat"}; if (wordBreak("catsandog", d)) { std::puts("case3"); return 1; } }
    { vector<string> d{"a"}; if (!wordBreak("a", d)) { std::puts("case4"); return 1; } }
    { vector<string> d{"aaaa","aaa"}; if (!wordBreak("aaaaaaa", d)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let dp[i] mean the prefix s[0..i) is fully segmentable. It holds when there is a cut point j with dp[j] true and the remaining piece s[j..i) present in the dictionary, which is stored in a hash set for O(1) lookups. Filling dp from left to right costs O(n^2) substring checks, O(n) space beyond the set.
