## challenge: Valid Palindrome II
tags: two-pointers, string, greedy
track: faang
difficulty: medium

Given a string `s`, return `true` if it can be a palindrome after deleting at most one character. Deleting zero characters is allowed, so any already-palindromic string qualifies.

Constraints: `1 <= s.length <= 10^5`, `s` consists of lowercase English letters.

Example: `s = "aba"` → `true`. Example: `s = "abca"` → `true` (delete `'c'`). Example: `s = "abc"` → `false`.

hint: Check for a palindrome with two pointers from both ends; the interesting moment is the first mismatch.
hint: At the first mismatch you are allowed exactly one deletion — it must be either the left character or the right character.
hint: On a mismatch, test whether the substring with the left char skipped is a palindrome, or the one with the right char skipped; if either is, the answer is true.

```cpp
// starter
#include <string>
bool validPalindrome(std::string s);
```

```cpp
bool validPalindrome(std::string s) {
    auto isPal = [&](int l, int r) {
        while (l < r) {
            if (s[l] != s[r]) return false;
            ++l;
            --r;
        }
        return true;
    };
    int lo = 0, hi = (int)s.size() - 1;
    while (lo < hi) {
        if (s[lo] != s[hi])
            return isPal(lo + 1, hi) || isPal(lo, hi - 1);
        ++lo;
        --hi;
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
    if (validPalindrome("aba") != true) { std::puts("case1"); return 1; }
    if (validPalindrome("abca") != true) { std::puts("case2"); return 1; }
    if (validPalindrome("abc") != false) { std::puts("case3"); return 1; }
    if (validPalindrome("racecar") != true) { std::puts("case4"); return 1; }
    if (validPalindrome("abcdef") != false) { std::puts("case5"); return 1; }
    if (validPalindrome("deeee") != true) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Walk two pointers inward comparing mirrored characters. As long as they match, the string is palindromic so far. At the first mismatch you must spend your single deletion on one of the two offending characters, so recursively (or iteratively) verify whether skipping the left character yields a palindrome for the inner range, or whether skipping the right character does. Either success proves the string is one deletion away from a palindrome. The scan plus the single follow-up check are both linear, so the total is O(n) time and O(1) extra space.
