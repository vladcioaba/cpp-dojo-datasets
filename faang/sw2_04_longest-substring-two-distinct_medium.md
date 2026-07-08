## challenge: Longest Substring with At Most Two Distinct Characters
tags: string, hash-table, sliding-window
track: faang
difficulty: medium

Given a string `s`, return the length of the longest contiguous substring that contains at most two distinct characters.

Constraints: `1 <= s.length <= 10^5`, `s` consists of English letters.

Example: `s = "eceba"` → `3` (the substring `"ece"` uses only `e` and `c`). Example: `s = "ccaabbb"` → `5` (the substring `"aabbb"`). Example: `s = "abaccc"` → `4` (the substring `"accc"`).

hint: Maintain a window plus a count of how many distinct characters it currently holds.

hint: Extend the right edge and update the frequency of the incoming character; the distinct count rises only when a brand-new character appears.

hint: Whenever the window holds more than two distinct characters, shrink from the left, decrementing frequencies until a character's count hits zero and the distinct count drops back to two.

```cpp
// starter
#include <string>
int lengthOfLongestSubstringTwoDistinct(std::string s);
```

```cpp
int lengthOfLongestSubstringTwoDistinct(std::string s) {
    std::array<int, 128> cnt{};
    int distinct = 0, left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        if (cnt[(unsigned char)s[right]]++ == 0) ++distinct;
        while (distinct > 2) {
            if (--cnt[(unsigned char)s[left]] == 0) --distinct;
            ++left;
        }
        best = std::max(best, right - left + 1);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <array>
#include <algorithm>
using std::string;
//__USER__
int main() {
    if (lengthOfLongestSubstringTwoDistinct("eceba")!=3) { std::puts("case1"); return 1; }
    if (lengthOfLongestSubstringTwoDistinct("ccaabbb")!=5) { std::puts("case2"); return 1; }
    if (lengthOfLongestSubstringTwoDistinct("abaccc")!=4) { std::puts("case3"); return 1; }
    if (lengthOfLongestSubstringTwoDistinct("a")!=1) { std::puts("case4"); return 1; }
    if (lengthOfLongestSubstringTwoDistinct("abcabcabc")!=2) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Keep a sliding window and a small frequency table indexed by character. As the right edge advances, bump the incoming character's count; a fresh character (count moving from 0 to 1) increases the distinct total. When the window exceeds two distinct characters, retract the left edge, decrementing counts, and drop the distinct total whenever a count returns to zero. The widest window observed is the answer. Since the alphabet is bounded, every update is O(1) and the scan is O(n) time, O(1) space.
