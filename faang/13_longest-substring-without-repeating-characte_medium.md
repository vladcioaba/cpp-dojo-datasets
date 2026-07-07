## challenge: Longest Substring Without Repeating Characters
tags: sliding-window, string, hash-table
track: faang
difficulty: medium

Given a string `s`, return the length of the longest substring (contiguous) without repeating characters.

Constraints: `0 <= s.length <= 5*10^4`, `s` consists of English letters, digits, symbols and spaces.

Example: `"abcabcbb"` → `3` (`"abc"`). Example: `"bbbbb"` → `1`. Example: `"pwwkew"` → `3` (`"wke"`). Example: `""` → `0`.

hint: Grow a window to the right; when a character repeats, the window's left edge must jump past its previous occurrence.
hint: A sliding window plus a table storing each character's last-seen index.

```cpp
// starter
#include <string>
int lengthOfLongestSubstring(std::string s);
```

```cpp
int lengthOfLongestSubstring(std::string s) {
    std::vector<int> last(256, -1);
    int start = 0, best = 0;
    for (int i = 0; i < (int)s.size(); ++i) {
        unsigned char c = (unsigned char)s[i];
        if (last[c] >= start) start = last[c] + 1;
        last[c] = i;
        best = std::max(best, i - start + 1);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <algorithm>
//__USER__
int main() {
    if (lengthOfLongestSubstring("abcabcbb") != 3) { std::puts("case1"); return 1; }
    if (lengthOfLongestSubstring("bbbbb")    != 1) { std::puts("case2"); return 1; }
    if (lengthOfLongestSubstring("pwwkew")   != 3) { std::puts("case3"); return 1; }
    if (lengthOfLongestSubstring("")         != 0) { std::puts("case4"); return 1; }
    if (lengthOfLongestSubstring(" ")        != 1) { std::puts("case5"); return 1; }
    if (lengthOfLongestSubstring("dvdf")     != 3) { std::puts("case6"); return 1; }
    if (lengthOfLongestSubstring("abba")     != 2) { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Slide a window while recording the last index at which each character appeared. When the current character was last seen at or after the window start, jump the start just past it; the answer is the widest window observed. O(n) time, O(min(n, alphabet)) space.
