## challenge: Arranging Coins
tags: binary-search, math
track: faang
difficulty: easy

You have `n` coins and want to build a staircase where the `i`-th row contains exactly `i` coins. The last row may be incomplete. Return the number of **complete** rows you can build.

Constraints: `1 <= n <= 2^31 - 1`.

Example: `n = 5` → `2` (rows of 1 and 2 use 3 coins; the third row is incomplete). Example: `n = 8` → `3`. Example: `n = 1` → `1`.

hint: `k` complete rows need `k * (k + 1) / 2` coins, which grows monotonically in `k` — search for the largest feasible `k`.
hint: Binary search `k` in `[0, n]`; a candidate is feasible when `k * (k + 1) / 2 <= n`.
hint: `k * (k + 1)` overflows 32 bits, and to bias toward the largest valid `k` use an upper mid `lo + (hi - lo + 1) / 2`.

```cpp
// starter
int arrangeCoins(int n);
```

```cpp
int arrangeCoins(int n) {
    long long lo = 0, hi = n;
    while (lo < hi) {
        long long mid = lo + (hi - lo + 1) / 2;   // upper mid, biased high
        if (mid * (mid + 1) / 2 <= (long long)n) lo = mid;
        else hi = mid - 1;
    }
    return (int)lo;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (arrangeCoins(5)          != 2)     { std::puts("case1"); return 1; }
    if (arrangeCoins(8)          != 3)     { std::puts("case2"); return 1; }
    if (arrangeCoins(1)          != 1)     { std::puts("case3"); return 1; }
    if (arrangeCoins(3)          != 2)     { std::puts("case4"); return 1; }
    if (arrangeCoins(2147483647) != 65535) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The coins used by `k` complete rows is the triangular number `k * (k + 1) / 2`, which increases with `k`, so "`k` rows fit in `n` coins" is a monotone predicate. Binary search the largest `k` satisfying it, using an upper mid so the window converges toward the maximum. Because `k * (k + 1)` overflows a 32-bit `int`, compute in `long long`. O(log n) time, O(1) space. (A closed form `floor((sqrt(8n + 1) - 1) / 2)` also works but risks floating-point rounding.)
