## challenge: Excel Sheet Column Number
tags: math, string
track: faang
difficulty: easy

Given a string `columnTitle` that represents the column title as it appears in an Excel sheet, return its corresponding column number. The columns are labeled `A, B, ..., Z, AA, AB, ..., AZ, BA, ...`, so `A -> 1`, `B -> 2`, `Z -> 26`, `AA -> 27`, and so on. This is essentially a base-26 numeral system in which the digits run from 1 (`A`) to 26 (`Z`) with no zero digit.

Constraints: `1 <= columnTitle.length <= 7`, `columnTitle` consists only of uppercase English letters, and it is between `A` and `FXSHRXW` (which maps to `2^31 - 1`).

Example: `columnTitle = "A"` → `1`. Example: `columnTitle = "AB"` → `28`. Example: `columnTitle = "ZY"` → `701`.

hint: This is a base-26 conversion where `A` is the digit 1 and `Z` is the digit 26.
hint: Scan left to right, and for each letter do `result = result * 26 + (c - 'A' + 1)`.
hint: The largest valid title `FXSHRXW` maps to `2^31 - 1`, so the running total fits in a 32-bit int.

```cpp
// starter
#include <string>
int titleToNumber(std::string columnTitle);
```

```cpp
int titleToNumber(std::string columnTitle) {
    int result = 0;
    for (char c : columnTitle)
        result = result * 26 + (c - 'A' + 1);
    return result;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
//__USER__
int main() {
    if (titleToNumber("A") != 1)               { std::puts("case1"); return 1; }
    if (titleToNumber("AB") != 28)             { std::puts("case2"); return 1; }
    if (titleToNumber("ZY") != 701)            { std::puts("case3"); return 1; }
    if (titleToNumber("AA") != 27)             { std::puts("case4"); return 1; }
    if (titleToNumber("Z") != 26)              { std::puts("case5"); return 1; }
    if (titleToNumber("FXSHRXW") != 2147483647){ std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Excel column labels are a bijective base-26 system: the digits are `A..Z` valued `1..26`, and there is no zero. Reading the title from most significant letter to least, each step multiplies the accumulated value by 26 and adds the current letter's value `c - 'A' + 1` — the standard Horner evaluation of a positional numeral. The problem guarantees the result fits in a signed 32-bit integer. O(L) time in the title length, O(1) space.
