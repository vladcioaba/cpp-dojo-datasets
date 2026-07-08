## challenge: First Unique Character in a String
tags: string, hash-table, counting, arrays-hashing
track: faang
difficulty: easy

Given a string `s`, return the index of the first non-repeating character in it. If there is no such character, return `-1`.

Constraints: `1 <= s.length <= 10^5`, `s` consists of lowercase English letters.

Example: `s = "leetcode"` → `0` (`l` occurs once). Example: `s = "loveleetcode"` → `2` (`v` is the first character that appears once). Example: `s = "aabb"` → `-1`.

hint: Whether a character is unique depends only on its total count, which one pass can compute.
hint: Count every letter's frequency first, then make a second pass in original order.
hint: The first index whose character has a frequency of exactly one is the answer; if the second pass finds none, return -1.

```cpp
// starter
#include <string>
int firstUniqChar(std::string s);
```

```cpp
int firstUniqChar(std::string s) {
    int cnt[26] = {0};
    for (char c : s) cnt[c - 'a']++;
    for (int i = 0; i < (int)s.size(); ++i)
        if (cnt[s[i] - 'a'] == 1) return i;
    return -1;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (firstUniqChar("leetcode") != 0) { std::puts("case1"); return 1; }
    if (firstUniqChar("loveleetcode") != 2) { std::puts("case2"); return 1; }
    if (firstUniqChar("aabb") != -1) { std::puts("case3"); return 1; }
    if (firstUniqChar("z") != 0) { std::puts("case4"); return 1; }
    if (firstUniqChar("cc") != -1) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Two linear passes suffice. The first tallies each letter's frequency into a size-26 array; the second walks the string in its original order and returns the index of the first letter whose count is one. Preserving the original scan order in the second pass guarantees "first" uniqueness. O(n) time, O(1) space.
