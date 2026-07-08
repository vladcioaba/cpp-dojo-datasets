## challenge: Sum of Square Numbers
tags: two-pointers, math, binary-search
track: faang
difficulty: medium

Given a non-negative integer `c`, decide whether there exist two non-negative integers `a` and `b` such that `a*a + b*b == c`. Return `true` if such a pair exists and `false` otherwise.

Constraints: `0 <= c <= 2^31 - 1`.

Example: `c = 5` → `true` (`1*1 + 2*2 = 5`). Example: `c = 3` → `false`. Example: `c = 4` → `true` (`0*0 + 2*2 = 4`).

hint: Any valid `a` and `b` satisfy `0 <= a, b <= sqrt(c)`, which bounds the search to that window.
hint: Put one pointer at `0` and the other at `floor(sqrt(c))`, and steer their squared sum toward `c`.
hint: If the sum is too small increase the low pointer, if too large decrease the high one; use 64-bit arithmetic so the squares do not overflow.

```cpp
// starter
bool judgeSquareSum(int c);
```

```cpp
bool judgeSquareSum(int c) {
    long long lo = 0, hi = (long long)std::sqrt((double)c);
    while (lo <= hi) {
        long long s = lo * lo + hi * hi;
        if (s == c) return true;
        else if (s < c) ++lo;
        else --hi;
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <cmath>
//__USER__
int main() {
    if (judgeSquareSum(5) != true) { std::puts("case1"); return 1; }
    if (judgeSquareSum(3) != false) { std::puts("case2"); return 1; }
    if (judgeSquareSum(4) != true) { std::puts("case3"); return 1; }
    if (judgeSquareSum(2) != true) { std::puts("case4"); return 1; }
    if (judgeSquareSum(0) != true) { std::puts("case5"); return 1; }
    if (judgeSquareSum(11) != false) { std::puts("case6"); return 1; }
    if (judgeSquareSum(100) != true) { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Both unknowns lie in `[0, sqrt(c)]`, so search that interval with two pointers `lo = 0` and `hi = floor(sqrt(c))`. Evaluate `lo*lo + hi*hi`: if it equals `c` you are done; if it is below `c` the sum can only grow by raising `lo`; if above, shrink it by lowering `hi`. Each pointer moves monotonically, so the scan is O(sqrt(c)) time and O(1) space. Compute the squares in 64-bit to avoid overflow near the constraint's upper limit.
