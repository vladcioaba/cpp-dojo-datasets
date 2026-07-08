## challenge: Substring with Concatenation of All Words
tags: string, hash-table, sliding-window, arrays-hashing
track: faang
difficulty: hard

You are given a string `s` and an array `words` of strings that all share the same length. A "concatenation substring" is any substring of `s` formed by joining every word in `words` exactly once, in any order, with no characters in between. Return the starting indices of all such substrings, in any order.

Constraints: `1 <= s.length <= 10^4`, `1 <= words.length <= 5000`, `1 <= words[i].length <= 30`, every `words[i]` has the same length, and `s` and `words[i]` consist of lowercase English letters.

Example: `s = "barfoothefoobarman", words = ["foo","bar"]` → `[0,9]` (`"barfoo"` at `0` and `"foobar"` at `9`). Example: `s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"]` → `[]` (no window uses `"word"` twice and `"good"`/`"best"` once).

hint: Every candidate window has the fixed length `words.length * words[0].length`, and it decomposes cleanly into consecutive word-sized chunks.
hint: Build a multiset (hash map of counts) of the required words, then slide across `s` in steps of one word length, comparing chunk counts.
hint: There are only `wordLen` distinct chunk alignments; run a sliding window over each alignment, shrinking from the left whenever a word appears too often, and record a hit whenever the window holds all words.

```cpp
// starter
#include <vector>
#include <string>
std::vector<int> findSubstring(std::string s, std::vector<std::string>& words);
```

```cpp
std::vector<int> findSubstring(std::string s, std::vector<std::string>& words) {
    std::vector<int> result;
    int wordLen = (int)words[0].size();
    int wordCount = (int)words.size();
    int n = (int)s.size();
    int span = wordLen * wordCount;
    if (n < span) return result;

    std::unordered_map<std::string, int> need;
    for (auto& w : words) ++need[w];

    for (int offset = 0; offset < wordLen; ++offset) {
        std::unordered_map<std::string, int> window;
        int left = offset, matched = 0;
        for (int right = offset; right + wordLen <= n; right += wordLen) {
            std::string word = s.substr(right, wordLen);
            auto it = need.find(word);
            if (it == need.end()) {
                window.clear();
                matched = 0;
                left = right + wordLen;
                continue;
            }
            ++window[word];
            ++matched;
            while (window[word] > it->second) {
                std::string leftWord = s.substr(left, wordLen);
                --window[leftWord];
                left += wordLen;
                --matched;
            }
            if (matched == wordCount) {
                result.push_back(left);
                std::string leftWord = s.substr(left, wordLen);
                --window[leftWord];
                left += wordLen;
                --matched;
            }
        }
    }
    return result;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using std::vector;
using std::string;
//__USER__
static bool eq(vector<int> got, vector<int> want) {
    std::sort(got.begin(), got.end());
    std::sort(want.begin(), want.end());
    return got == want;
}
int main() {
    { string s = "barfoothefoobarman"; vector<string> w{"foo","bar"};
      if (!eq(findSubstring(s, w), {0,9})) { std::puts("case1"); return 1; } }
    { string s = "wordgoodgoodgoodbestword"; vector<string> w{"word","good","best","word"};
      if (!eq(findSubstring(s, w), {})) { std::puts("case2"); return 1; } }
    { string s = "barfoofoobarthefoobarman"; vector<string> w{"bar","foo","the"};
      if (!eq(findSubstring(s, w), {6,9,12})) { std::puts("case3"); return 1; } }
    { string s = "aaa"; vector<string> w{"a","a"};
      if (!eq(findSubstring(s, w), {0,1})) { std::puts("case4"); return 1; } }
    { string s = "foobar"; vector<string> w{"bar"};
      if (!eq(findSubstring(s, w), {3})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Since all words share a length `L`, any concatenation occupies a fixed `L * words.size()` span made of word-sized chunks. We only need to consider `L` starting alignments; within each, a sliding window advances one word at a time, maintaining a count map and shrinking from the left when a word exceeds its required multiplicity. A window that matches every word yields a start index. Each alignment scans `s` once, giving O(n * L) time overall and O(words.size()) space.
