## challenge: Pow(x, n)
tags: math, divide-and-conquer, bit-tricks
track: faang
difficulty: medium

Implement `pow(x, n)`, which raises `x` to the integer power `n` (that is, computes `x^n`). Multiplying `x` by itself `n` times is too slow for large `n`; use exponentiation by squaring.

Constraints: `-100.0 < x < 100.0`, `-2^31 <= n <= 2^31 - 1`, the result is guaranteed to fit within a `double` (relative error within `1e-6` is accepted).

Example: `x = 2.0, n = 10` → `1024.0`. Example: `x = 2.0, n = -2` → `0.25`. Example: `x = 2.1, n = 3` → `9.261`.

hint: `x^n` can be built from `x^(n/2)`: square the half-power, and multiply by one extra `x` if `n` is odd.
hint: Handle negative exponents by inverting the base (`x = 1/x`) and negating `n`. Widen `n` to 64 bits first so that negating `-2^31` does not overflow.
hint: Iterate over the bits of `n`: whenever the current low bit is 1 multiply it into the result, then square `x` and shift `n` right.

```cpp
// starter
double myPow(double x, int n);
```

```cpp
double myPow(double x, int n) {
    long long e = n;
    if (e < 0) { x = 1.0 / x; e = -e; }
    double result = 1.0;
    while (e > 0) {
        if (e & 1) result *= x;
        x *= x;
        e >>= 1;
    }
    return result;
}
```

```cpp
// harness
#include <cstdio>
#include <cmath>
//__USER__
static bool close(double a, double b) { return std::fabs(a - b) < 1e-6; }
int main() {
    if (!close(myPow(2.0, 10), 1024.0))  { std::puts("case1"); return 1; }
    if (!close(myPow(2.0, -2), 0.25))    { std::puts("case2"); return 1; }
    if (!close(myPow(2.1, 3), 9.261))    { std::puts("case3"); return 1; }
    if (!close(myPow(1.0, -2147483648), 1.0)) { std::puts("case4"); return 1; }
    if (!close(myPow(2.0, 0), 1.0))      { std::puts("case5"); return 1; }
    if (!close(myPow(0.5, 2), 0.25))     { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Exponentiation by squaring reads the exponent in binary. Each squaring of `x` produces `x^1, x^2, x^4, ...`, and multiplying the running result by the current power exactly when that bit of `n` is set assembles `x^n` in O(log n) multiplications. Negative exponents are reduced by inverting the base once and working with `|n|`; promoting `n` to a 64-bit `long long` before negation sidesteps the overflow of `-(-2^31)`. O(log n) time, O(1) space.
