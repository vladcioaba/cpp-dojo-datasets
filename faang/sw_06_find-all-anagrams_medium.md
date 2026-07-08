## challenge: Find All Anagrams in a String
tags: string, hash-table, sliding-window
track: faang
difficulty: medium

Given two strings `s` and `p`, return the start indices (in ascending order) of every substring of `s` that is an anagram of `p`. An anagram uses exactly the same letters with the same multiplicities, in any order.

Constraints: `1 <= s.length, p.length <= 3 * 10^4`, both strings consist of lowercase English letters.

Example: `s = "cbaebabacd", p = "abc"` → `[0,6]` (`"cba"` at 0, `"bac"` at 6). Example: `s = "abab", p = "ab"` → `[0,1,2]`.

hint: Every anagram of `p` in `s` is a window of length `p.length` whose letter frequencies match those of `p`.
hint: Two length-26 frequency arrays — one fixed for `p`, one for the current window — let you compare in constant time.
hint: Slide the window by one: increment the entering letter and decrement the leaving one, recording the start index whenever the arrays match.

```cpp
// starter
#include <string>
#include <vector>
std::vector<int> findAnagrams(std::string s, std::string p);
```

```cpp
std::vector<int> findAnagrams(std::string s, std::string p) {
    std::vector<int> res;
    int n = s.size(), m = p.size();
    if (n < m) return res;
    std::array<int,26> need{}, win{};
    for (char c : p) ++need[c - 'a'];
    for (int i = 0; i < n; ++i) {
        ++win[s[i] - 'a'];
        if (i >= m) --win[s[i - m] - 'a'];
        if (i >= m - 1 && win == need) res.push_back(i - m + 1);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <array>
using std::vector;
using std::string;
//__USER__
static bool eq(const vector<int>& a, const vector<int>& b) {
    if (a.size() != b.size()) return false;
    for (size_t i = 0; i < a.size(); ++i) if (a[i] != b[i]) return false;
    return true;
}
int main() {
    if (!eq(findAnagrams("cbaebabacd","abc"), {0,6})) { std::puts("case1"); return 1; }
    if (!eq(findAnagrams("abab","ab"), {0,1,2})) { std::puts("case2"); return 1; }
    if (!eq(findAnagrams("aa","bb"), {})) { std::puts("case3"); return 1; }
    if (!eq(findAnagrams("baa","aa"), {1})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Fix the window length to `p.length`. Precompute the letter counts of `p` in a length-26 array, and maintain the same counts for the current window as it slides: add the entering character on the right and drop the character leaving on the left. Whenever the two count arrays are equal, the window is an anagram, so push its start index. Comparing two 26-slot arrays is O(1), so the whole scan is O(n) time and O(1) space.
