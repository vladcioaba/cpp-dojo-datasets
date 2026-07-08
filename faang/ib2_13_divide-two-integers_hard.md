## challenge: Divide Two Integers
tags: bit-tricks, math
track: faang
difficulty: hard

Given two integers `dividend` and `divisor`, divide them without using multiplication, division, or the mod operator. The integer division should truncate toward zero, so `8.345` becomes `8` and `-2.7` becomes `-2`. Return the quotient. Because the environment stores signed 32-bit integers, if the quotient is strictly greater than `2^31 - 1` return `2^31 - 1`, and if it is strictly less than `-2^31` return `-2^31`; the only case that overflows is `dividend = -2^31, divisor = -1`.

Constraints: `-2^31 <= dividend, divisor <= 2^31 - 1`, `divisor != 0`.

Example: `dividend = 10, divisor = 3` → `3`. Example: `dividend = 7, divisor = -3` → `-2`. Example: `dividend = -2147483648, divisor = -1` → `2147483647`.

hint: Work with 64-bit magnitudes to avoid overflow when negating `-2^31`, and restore the sign at the end.
hint: Repeatedly subtract the largest shifted multiple `divisor << k` that still fits, adding `1 << k` to the quotient — this is long division in binary.
hint: Only `-2^31 / -1` overflows the signed 32-bit range; special-case it to `2^31 - 1`.

```cpp
// starter
int divide(int dividend, int divisor);
```

```cpp
int divide(int dividend, int divisor) {
    if (dividend == INT_MIN && divisor == -1) return INT_MAX;
    long long a = dividend, b = divisor;
    bool neg = (a < 0) != (b < 0);
    if (a < 0) a = -a;
    if (b < 0) b = -b;
    long long quotient = 0;
    for (int k = 31; k >= 0; --k) {
        if ((a >> k) >= b) {
            a -= b << k;
            quotient += 1LL << k;
        }
    }
    return (int)(neg ? -quotient : quotient);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    if (divide(10, 3) != 3)                        { std::puts("case1"); return 1; }
    if (divide(7, -3) != -2)                       { std::puts("case2"); return 1; }
    if (divide(-2147483648, -1) != 2147483647)     { std::puts("case3"); return 1; }
    if (divide(-2147483648, 1) != -2147483648)     { std::puts("case4"); return 1; }
    if (divide(0, 1) != 0)                         { std::puts("case5"); return 1; }
    if (divide(-7, 2) != -3)                       { std::puts("case6"); return 1; }
    if (divide(-2147483648, 2) != -1073741824)     { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Determine the result sign from the operands, then compute with non-negative 64-bit magnitudes so that negating `-2^31` is safe. Binary long division tries each power of two from `2^31` down to `2^0`: whenever `divisor << k` still fits inside the remaining dividend (`(a >> k) >= b`), subtract it and add `1 << k` to the quotient. This assembles the quotient bit by bit using only shifts, additions, and subtractions. The single genuine 32-bit overflow, `-2^31 / -1`, is clamped to `2^31 - 1`. O(32) iterations, O(1) space.
