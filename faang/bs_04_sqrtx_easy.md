## challenge: Sqrt(x)
tags: binary-search, math
track: faang
difficulty: easy

Given a non-negative integer `x`, return the integer square root of `x` — that is, the largest integer `r` such that `r * r <= x`. You may not use any built-in exponent or square-root function. The fractional part is truncated.

Constraints: `0 <= x <= 2^31 - 1`.

Example: `x = 4` → `2`. Example: `x = 8` → `2` (since `sqrt(8) ≈ 2.828`, truncated). Example: `x = 0` → `0`. Example: `x = 1` → `1`.

hint: `r * r` is monotonically increasing in `r`, so the set of `r` with `r * r <= x` is a prefix — binary search for its last element.
hint: Search `r` in `[1, x]`; the answer is the greatest `r` whose square does not exceed `x`.
hint: `r * r` can exceed 32-bit range, so compute the product in 64-bit (`long long`) before comparing to `x`.

```cpp
// starter
int mySqrt(int x);
```

```cpp
int mySqrt(int x) {
    if (x < 2) return x;
    long long lo = 1, hi = x;
    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (mid * mid <= (long long)x) lo = mid + 1;
        else hi = mid - 1;
    }
    return (int)hi;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (mySqrt(4) != 2)                { std::puts("case1"); return 1; }
    if (mySqrt(8) != 2)                { std::puts("case2"); return 1; }
    if (mySqrt(0) != 0)                { std::puts("case3"); return 1; }
    if (mySqrt(1) != 1)                { std::puts("case4"); return 1; }
    if (mySqrt(2147483647) != 46340)   { std::puts("case5"); return 1; }
    if (mySqrt(2147395600) != 46340)   { std::puts("case6"); return 1; }
    if (mySqrt(2147395599) != 46339)   { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Binary search on the answer. The predicate `r * r <= x` holds for a prefix `1..r*`, so we search for the largest such `r`. When the loop exits, `hi` sits on that last feasible value. The multiplication is done in 64-bit to avoid overflow near `x = 2^31 - 1`, where the true root is `46340`. O(log x) time, O(1) space.
