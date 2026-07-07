## challenge: Valid Anagram
tags: string, hash-table, sorting
track: faang
difficulty: easy

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise. An anagram uses exactly the same letters with the same multiplicities, just reordered.

Constraints: `0 <= s.length, t.length <= 5 * 10^4`, both consist of lowercase English letters.

Example: `s = "anagram", t = "nagaram"` → `true`. Example: `s = "rat", t = "car"` → `false`. Example: `s = "", t = ""` → `true`.

hint: Anagrams must have identical letter frequencies, so length must match first.
hint: For lowercase letters a fixed-size array of 26 counts is enough — no map needed.
hint: Add one for each character of `s`, subtract one for each character of `t`; every count must end at zero.

```cpp
// starter
#include <string>
bool isAnagram(std::string s, std::string t);
```

```cpp
bool isAnagram(std::string s, std::string t) {
    if (s.size() != t.size()) return false;
    std::array<int, 26> count{};
    for (char c : s) ++count[c - 'a'];
    for (char c : t) --count[c - 'a'];
    for (int v : count) if (v != 0) return false;
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <array>
using std::string;
//__USER__
int main() {
    if (!isAnagram("anagram", "nagaram")) { std::puts("case1"); return 1; }
    if ( isAnagram("rat", "car"))         { std::puts("case2"); return 1; }
    if ( isAnagram("a", "ab"))            { std::puts("case3"); return 1; }
    if ( isAnagram("aacc", "ccac"))       { std::puts("case4"); return 1; }
    if (!isAnagram("", ""))               { std::puts("case5"); return 1; }
    if (!isAnagram("listen", "silent"))   { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** If the lengths differ the answer is immediately `false`. Otherwise tally letter frequencies in a size-26 array — increment for `s`, decrement for `t` — and the strings are anagrams exactly when every counter returns to zero. This is O(n) time and O(1) extra space, faster than the O(n log n) approach of sorting both strings.
