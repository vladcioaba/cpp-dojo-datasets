## challenge: Roman to Integer
tags: math, string, hash-table
track: faang
difficulty: easy

Roman numerals are formed from the symbols `I=1, V=5, X=10, L=50, C=100, D=500, M=1000`. Usually symbols are written largest to smallest and added, but six subtractive pairs exist: `IV=4`, `IX=9`, `XL=40`, `XC=90`, `CD=400`, and `CM=900`. Given a valid Roman numeral string `s`, convert it to its integer value.

Constraints: `1 <= s.length <= 15`, `s` contains only the characters `I, V, X, L, C, D, M`, and `s` is guaranteed to be a valid Roman numeral in the range `[1, 3999]`.

Example: `s = "III"` → `3`. Example: `s = "LVIII"` → `58`. Example: `s = "MCMXCIV"` → `1994`.

hint: Assign each symbol its value, then decide per symbol whether to add or subtract.
hint: A symbol is subtracted exactly when a strictly larger symbol follows it (e.g. the `I` in `IV`).
hint: Sweep left to right: if `value(s[i]) < value(s[i+1])`, subtract it, otherwise add it.

```cpp
// starter
#include <string>
int romanToInt(std::string s);
```

```cpp
int romanToInt(std::string s) {
    auto val = [](char c) {
        switch (c) {
            case 'I': return 1;
            case 'V': return 5;
            case 'X': return 10;
            case 'L': return 50;
            case 'C': return 100;
            case 'D': return 500;
            case 'M': return 1000;
        }
        return 0;
    };
    int total = 0, n = (int)s.size();
    for (int i = 0; i < n; ++i) {
        if (i + 1 < n && val(s[i]) < val(s[i + 1])) total -= val(s[i]);
        else total += val(s[i]);
    }
    return total;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
//__USER__
int main() {
    if (romanToInt("III") != 3)          { std::puts("case1"); return 1; }
    if (romanToInt("LVIII") != 58)       { std::puts("case2"); return 1; }
    if (romanToInt("MCMXCIV") != 1994)   { std::puts("case3"); return 1; }
    if (romanToInt("IV") != 4)           { std::puts("case4"); return 1; }
    if (romanToInt("IX") != 9)           { std::puts("case5"); return 1; }
    if (romanToInt("MMMCMXCIX") != 3999) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Map each symbol to its value. In a valid numeral a symbol represents a subtraction precisely when a larger symbol immediately follows it (the two subtractive characters like `IX` are the only places a smaller value precedes a larger one). So a single left-to-right pass suffices: subtract the current value when the next symbol is larger, otherwise add it. This naturally handles both additive runs and the six subtractive pairs without special-casing. O(n) time, O(1) space.
