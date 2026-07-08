## challenge: Longest Substring with At Most K Distinct Characters
tags: string, hash-table, sliding-window
track: faang
difficulty: medium

Given a string `s` and an integer `k`, return the length of the longest contiguous substring that contains at most `k` distinct characters.

Constraints: `1 <= s.length <= 5 * 10^4`, `0 <= k <= 50`, `s` consists of English letters.

Example: `s = "eceba", k = 2` → `3` (the substring `"ece"`). Example: `s = "aa", k = 1` → `2`. Example: `s = "abcadcacacaca", k = 3` → `11` (the substring `"cadcacacaca"`).

hint: This generalizes the "at most two distinct" window — track the current number of distinct characters against the limit `k`.

hint: Expand the right edge and update a frequency table; the distinct count grows only when a character first appears in the window.

hint: While the window holds more than `k` distinct characters, shrink from the left until one character's count reaches zero and the distinct count falls back within the limit. Guard the degenerate `k == 0` case.

```cpp
// starter
#include <string>
int lengthOfLongestSubstringKDistinct(std::string s, int k);
```

```cpp
int lengthOfLongestSubstringKDistinct(std::string s, int k) {
    if (k == 0) return 0;
    std::array<int, 128> cnt{};
    int distinct = 0, left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        if (cnt[(unsigned char)s[right]]++ == 0) ++distinct;
        while (distinct > k) {
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
    if (lengthOfLongestSubstringKDistinct("eceba",2)!=3) { std::puts("case1"); return 1; }
    if (lengthOfLongestSubstringKDistinct("aa",1)!=2) { std::puts("case2"); return 1; }
    if (lengthOfLongestSubstringKDistinct("abcadcacacaca",3)!=11) { std::puts("case3"); return 1; }
    if (lengthOfLongestSubstringKDistinct("a",0)!=0) { std::puts("case4"); return 1; }
    if (lengthOfLongestSubstringKDistinct("aba",1)!=1) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A single variable-width window solves the whole family of "at most `k` distinct" problems. Extend the right edge, updating a frequency table and incrementing the distinct count only when a character appears for the first time in the window; when the distinct count exceeds `k`, retract the left edge, decrementing counts and lowering the distinct total each time a count returns to zero. Track the maximum valid width. Because the alphabet is fixed, every step is O(1), giving O(n) time and O(1) space. Handle `k == 0` up front, where no non-empty window is allowed.
