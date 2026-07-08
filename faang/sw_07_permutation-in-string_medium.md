## challenge: Permutation in String
tags: string, hash-table, sliding-window
track: faang
difficulty: medium

Given two strings `s1` and `s2`, return `true` if `s2` contains a substring that is a permutation of `s1`, and `false` otherwise. Equivalently, one of `s1`'s permutations appears as a contiguous block of `s2`.

Constraints: `1 <= s1.length, s2.length <= 10^4`, both strings consist of lowercase English letters.

Example: `s1 = "ab", s2 = "eidbaooo"` → `true` (`"ba"`). Example: `s1 = "ab", s2 = "eidboaoo"` → `false`.

hint: A permutation of `s1` is exactly a window in `s2` of length `s1.length` whose letter counts equal those of `s1`.
hint: Maintain a length-26 count for the current fixed-size window and compare it against the counts of `s1`.
hint: Slide the window one step at a time, updating the entering and leaving letters, and return `true` the first time the counts match.

```cpp
// starter
#include <string>
bool checkInclusion(std::string s1, std::string s2);
```

```cpp
bool checkInclusion(std::string s1, std::string s2) {
    int n = s1.size(), m = s2.size();
    if (n > m) return false;
    std::array<int,26> need{}, win{};
    for (char c : s1) ++need[c - 'a'];
    for (int i = 0; i < m; ++i) {
        ++win[s2[i] - 'a'];
        if (i >= n) --win[s2[i - n] - 'a'];
        if (i >= n - 1 && win == need) return true;
    }
    return false;
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
    if (checkInclusion("ab","eidbaooo") != true) { std::puts("case1"); return 1; }
    if (checkInclusion("ab","eidboaoo") != false) { std::puts("case2"); return 1; }
    if (checkInclusion("adc","dcda") != true) { std::puts("case3"); return 1; }
    if (checkInclusion("hello","ooolleoooleh") != false) { std::puts("case4"); return 1; }
    if (checkInclusion("a","a") != true) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A permutation match is a window of length `s1.length` in `s2` whose letter frequencies equal those of `s1`. Build `s1`'s counts once, then slide a fixed-width window across `s2`, incrementing the entering letter and decrementing the one that leaves. As soon as the window counts equal `s1`'s counts, a permutation exists. Each 26-slot comparison is O(1), so the algorithm runs in O(n) time and O(1) space.
