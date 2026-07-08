## challenge: Longest Repeating Character Replacement
tags: string, sliding-window
track: faang
difficulty: hard

Given a string `s` of uppercase English letters and an integer `k`, you may replace at most `k` characters with any uppercase letters. Return the length of the longest substring that can be made to contain a single repeated letter after performing at most `k` such replacements.

Constraints: `1 <= s.length <= 10^5`, `s` consists of uppercase English letters, `0 <= k <= s.length`.

Example: `s = "ABAB", k = 2` → `4` (replace the two `A`s with `B`s, or vice versa). Example: `s = "AABABBA", k = 1` → `4` (window `"ABBA"` → change one character to get four identical letters).

hint: A window is fixable when the number of characters you must change — its length minus the count of its most frequent letter — is at most `k`.
hint: Track a running frequency table for the window and the highest single-letter count it has ever contained.
hint: When `windowLength - maxCount` exceeds `k`, slide the left edge forward by one; the largest window width the scan reaches is the answer.

```cpp
// starter
#include <string>
int characterReplacement(std::string s, int k);
```

```cpp
int characterReplacement(std::string s, int k) {
    std::array<int,26> cnt{};
    int left = 0, maxCount = 0, best = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        maxCount = std::max(maxCount, ++cnt[s[right] - 'A']);
        if (right - left + 1 - maxCount > k) {
            --cnt[s[left] - 'A'];
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
    if (characterReplacement("ABAB",2)!=4) { std::puts("case1"); return 1; }
    if (characterReplacement("AABABBA",1)!=4) { std::puts("case2"); return 1; }
    if (characterReplacement("AAAA",0)!=4) { std::puts("case3"); return 1; }
    if (characterReplacement("ABCDE",1)!=2) { std::puts("case4"); return 1; }
    if (characterReplacement("AAAB",0)!=3) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A window can be turned into a single repeated letter with at most `k` edits exactly when `windowLength - (count of its most frequent letter) <= k`. Grow the window to the right, updating a 26-slot frequency table and the best single-letter count `maxCount`. When the window becomes invalid, shift the left edge by one — note the window never needs to shrink further, because a smaller valid window can never beat one already recorded, so `maxCount` may safely stay stale. The largest width reached is the answer, in O(n) time and O(1) space.
