## challenge: Valid Palindrome
tags: two-pointers, string
track: faang
difficulty: easy

Given a string `s`, return `true` if it is a palindrome considering only alphanumeric characters and ignoring case. All other characters are skipped. An empty string (after filtering) counts as a palindrome.

Constraints: `1 <= s.length <= 2*10^5`, `s` consists of printable ASCII.

Example: `"A man, a plan, a canal: Panama"` → `true`. Example: `"race a car"` → `false`. Example: `" "` → `true`.

```cpp
// starter
#include <string>
bool isPalindrome(std::string s);
```

```cpp
bool isPalindrome(std::string s) {
    int i = 0, j = (int)s.size() - 1;
    auto alnum = [](char c){ return std::isalnum((unsigned char)c) != 0; };
    auto lower = [](char c){ return (char)std::tolower((unsigned char)c); };
    while (i < j) {
        while (i < j && !alnum(s[i])) ++i;
        while (i < j && !alnum(s[j])) --j;
        if (lower(s[i]) != lower(s[j])) return false;
        ++i; --j;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <cctype>
//__USER__
int main() {
    if (!isPalindrome("A man, a plan, a canal: Panama")) { std::puts("case1"); return 1; }
    if ( isPalindrome("race a car"))                     { std::puts("case2"); return 1; }
    if (!isPalindrome(" "))                              { std::puts("case3"); return 1; }
    if (!isPalindrome("0P0"))                            { std::puts("case4"); return 1; }
    if (!isPalindrome("aa"))                             { std::puts("case5"); return 1; }
    if ( isPalindrome("ab"))                             { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```
