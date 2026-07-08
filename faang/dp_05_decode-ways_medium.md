## challenge: Decode Ways
tags: dynamic-programming, string
track: faang
difficulty: medium

A message of digits was encoded from letters using the mapping `'A' -> "1"`, `'B' -> "2"`, ..., `'Z' -> "26"`. Given the encoded string `s`, return the number of ways it can be decoded back into letters. A group of digits with a leading zero (like `"06"`) is not a valid letter, and a lone `'0'` decodes to nothing.

Constraints: `1 <= s.length <= 100`, `s` contains only digits.

Example: `s = "226"` → `3` (`"BZ"`, `"VF"`, `"BBF"`). Example: `s = "06"` → `0` (no valid decoding).

hint: At each position you may consume one digit (if it is not `'0'`) or two digits (if they form a number in `[10, 26]`).
hint: Let `dp[i]` be the number of decodings of the first `i` characters; `dp[0] = 1`.
hint: `dp[i] = (single-digit valid ? dp[i-1] : 0) + (two-digit valid ? dp[i-2] : 0)`.

```cpp
// starter
#include <string>
int numDecodings(std::string s);
```

```cpp
int numDecodings(std::string s) {
    int n = (int)s.size();
    if (n == 0 || s[0] == '0') return 0;
    int prev = 1, cur = 1;  // dp[i-2], dp[i-1]
    for (int i = 1; i < n; ++i) {
        int tmp = 0;
        if (s[i] != '0') tmp += cur;
        int two = (s[i - 1] - '0') * 10 + (s[i] - '0');
        if (two >= 10 && two <= 26) tmp += prev;
        if (tmp == 0) return 0;
        prev = cur;
        cur = tmp;
    }
    return cur;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
//__USER__
int main() {
    if (numDecodings("12")   != 2) { std::puts("case1"); return 1; }
    if (numDecodings("226")  != 3) { std::puts("case2"); return 1; }
    if (numDecodings("06")   != 0) { std::puts("case3"); return 1; }
    if (numDecodings("10")   != 1) { std::puts("case4"); return 1; }
    if (numDecodings("2101") != 1) { std::puts("case5"); return 1; }
    if (numDecodings("27")   != 1) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Scan left to right keeping dp[i], the number of decodings of the first i characters. Each new digit contributes dp[i-1] ways when it is a valid single letter (not '0') and dp[i-2] ways when it pairs with its predecessor into a value in [10, 26]. Two rolling scalars give O(n) time and O(1) space; hitting a position with zero ways means the string is undecodable.
