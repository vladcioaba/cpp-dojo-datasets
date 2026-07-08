## challenge: Word Break II
tags: backtracking, string, dynamic-programming, memoization
track: faang
difficulty: hard

Given a string `s` and a dictionary of strings `wordDict`, add spaces to `s` to construct a sentence where each word is a valid dictionary word. Return all such possible sentences in any order. The same dictionary word may be reused any number of times in the segmentation.

Constraints: `1 <= s.length <= 20`, `1 <= wordDict.length <= 1000`, `1 <= wordDict[i].length <= 10`, `s` and `wordDict[i]` consist of lowercase English letters, all dictionary words are distinct.

Example: `s = "catsanddog", wordDict = ["cat","cats","and","sand","dog"]` -> `["cats and dog","cat sand dog"]`. Example: `s = "pineapplepenapple", wordDict = ["apple","pen","applepen","pine","pineapple"]` -> `["pine apple pen apple","pineapple pen apple","pine applepen apple"]`. Example: `s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]` -> `[]`.

hint: From a starting index, try every dictionary word that matches the prefix, then recursively segment the remainder and prepend the word to each sentence you get back.
hint: The suffix starting at a given index always produces the same set of sentences, so memoize results by start index to avoid recomputing shared suffixes.
hint: The base case is reaching the end of the string, which yields exactly one empty sentence; joining a word with an empty suffix just gives the word itself.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> wordBreak(std::string s, std::vector<std::string>& wordDict);
```

```cpp
std::vector<std::string> wordBreak(std::string s, std::vector<std::string>& wordDict) {
    std::unordered_set<std::string> dict(wordDict.begin(), wordDict.end());
    std::unordered_map<int, std::vector<std::string>> memo;
    int n = (int)s.size();
    std::function<std::vector<std::string>(int)> dfs = [&](int start) -> std::vector<std::string> {
        auto it = memo.find(start);
        if (it != memo.end()) return it->second;
        std::vector<std::string> result;
        if (start == n) { result.push_back(""); return memo[start] = result; }
        for (int end = start + 1; end <= n; ++end) {
            std::string word = s.substr(start, end - start);
            if (dict.count(word)) {
                for (const std::string& sub : dfs(end))
                    result.push_back(sub.empty() ? word : word + " " + sub);
            }
        }
        return memo[start] = result;
    };
    return dfs(0);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <unordered_set>
#include <unordered_map>
#include <functional>
#include <algorithm>
using std::vector;
using std::string;
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    {
        vector<string> dict = {"cat","cats","and","sand","dog"};
        vector<string> want = {"cats and dog","cat sand dog"};
        if (canon(wordBreak("catsanddog", dict)) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        vector<string> dict = {"apple","pen","applepen","pine","pineapple"};
        vector<string> want = {"pine apple pen apple","pineapple pen apple","pine applepen apple"};
        if (canon(wordBreak("pineapplepenapple", dict)) != canon(want)) { std::puts("case2"); return 1; }
    }
    {
        vector<string> dict = {"cats","dog","sand","and","cat"};
        if (!wordBreak("catsandog", dict).empty()) { std::puts("case3"); return 1; }
    }
    {
        vector<string> dict = {"a","aa","aaa"};
        auto got = wordBreak("aaa", dict);
        // aaa | aa a | a aa | a a a  -> 4 sentences
        if (got.size() != 4) { std::puts("size4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** For each start index, try every dictionary word matching the prefix and recurse on the remaining suffix, prepending the word to every returned sentence. Because the suffix from a given index always yields the same sentences, memoizing by start index collapses the shared work and prevents exponential re-exploration of common suffixes (the classic worst case). The base case at the end of the string returns a single empty sentence. Time is proportional to the number of sentences times their length, with the dictionary in a hash set for O(1) membership.
