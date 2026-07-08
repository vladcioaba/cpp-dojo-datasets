## challenge: Minimum Length of String After Deleting Similar Ends
tags: two-pointers, string
track: faang
difficulty: medium

Given a string `s` consisting only of the characters `a`, `b`, and `c`, you may repeatedly apply the following operation: pick a non-empty prefix and a non-empty suffix that are made of the *same* character, are disjoint, and delete both. Return the minimum possible length of the string after any number of such operations.

Constraints: `1 <= s.length <= 10^5`, `s[i]` is one of `'a'`, `'b'`, or `'c'`.

Example: `s = "ca"` → `2` (the ends differ, nothing can be removed). Example: `s = "cabaabac"` → `0`. Example: `s = "aabccabba"` → `3`.

hint: You can only ever peel from the two ends, so track a left and a right pointer bounding what remains.
hint: An operation is possible exactly when the character at the left pointer equals the character at the right pointer.
hint: When they match, consume the entire run of that character from both sides at once, then re-check the new ends.

```cpp
// starter
#include <string>
int minimumLength(std::string s);
```

```cpp
int minimumLength(std::string s) {
    int lo = 0, hi = (int)s.size() - 1;
    while (lo < hi && s[lo] == s[hi]) {
        char c = s[lo];
        while (lo <= hi && s[lo] == c) ++lo;
        while (hi >= lo && s[hi] == c) --hi;
    }
    return hi - lo + 1;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (minimumLength("ca") != 2) { std::puts("case1"); return 1; }
    if (minimumLength("cabaabac") != 0) { std::puts("case2"); return 1; }
    if (minimumLength("aabccabba") != 3) { std::puts("case3"); return 1; }
    if (minimumLength("a") != 1) { std::puts("case4"); return 1; }
    if (minimumLength("aa") != 0) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Since operations only remove from the two ends, maintain pointers `lo` and `hi` marking the surviving span. As long as they point at the same character, an operation is available: strip the whole contiguous run of that character from the left and, independently, from the right, because those characters can be paired off and deleted. Repeat until the ends differ or the pointers cross. The remaining length is `hi - lo + 1` (clamped to `0` when they cross). Each character is visited once, so O(n) time and O(1) space.
