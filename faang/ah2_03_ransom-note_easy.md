## challenge: Ransom Note
tags: string, hash-table, counting, arrays-hashing
track: faang
difficulty: easy

Given two strings `ransomNote` and `magazine`, return `true` if `ransomNote` can be constructed by using the letters from `magazine`. Each letter in `magazine` may be used at most once.

Constraints: `1 <= ransomNote.length, magazine.length <= 10^5`, both consist of lowercase English letters.

Example: `ransomNote = "a", magazine = "b"` → `false`. Example: `ransomNote = "aa", magazine = "ab"` → `false` (only one `a` available). Example: `ransomNote = "aa", magazine = "aab"` → `true`.

hint: You only need to know how many of each letter the magazine supplies, not their positions.
hint: Count the 26 letter frequencies in `magazine` first.
hint: Walk through `ransomNote` and decrement the corresponding count; if any count goes negative, the magazine is short a letter and you can return false.

```cpp
// starter
#include <string>
bool canConstruct(std::string ransomNote, std::string magazine);
```

```cpp
bool canConstruct(std::string ransomNote, std::string magazine) {
    int cnt[26] = {0};
    for (char c : magazine) cnt[c - 'a']++;
    for (char c : ransomNote) {
        if (--cnt[c - 'a'] < 0) return false;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (canConstruct("a", "b") != false) { std::puts("case1"); return 1; }
    if (canConstruct("aa", "ab") != false) { std::puts("case2"); return 1; }
    if (canConstruct("aa", "aab") != true) { std::puts("case3"); return 1; }
    if (canConstruct("bg", "efjbdfbdgfjhhaijndikhbougmarfdd") != true) { std::puts("case4"); return 1; }
    if (canConstruct("z", "zz") != true) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** This is a multiset containment check. Tally each letter available in `magazine` into a size-26 array, then consume letters as `ransomNote` demands them. If a demand ever pushes a count below zero the magazine lacks enough of that letter and construction is impossible. Linear in the combined length with constant extra space.
