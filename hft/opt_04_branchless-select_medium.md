## challenge: Branchless select (cond ? a : b)
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

A ternary `cond ? a : b` on unpredictable `cond` is a branch, and on the hot path an irregular condition mispredicts for ~15-20 cycles each time. Implement `int bselect(int cond, int a, int b)` that returns `a` when `cond` is nonzero and `b` when `cond` is `0`, using a bitmask select instead of a branch. This is the primitive every other branchless trick is built on.

Constraints: `a` and `b` are any 32-bit `int` including `INT_MIN`/`INT_MAX`; `cond` is any int treated as a boolean. No arithmetic on `a`/`b` beyond bitwise ops — so no overflow is possible.

Example: `bselect(1, 10, 20)` → `10`. Example: `bselect(0, 10, 20)` → `20`. Example: `bselect(5, -1, INT_MIN)` → `-1` (any nonzero `cond` selects `a`).

hint: Normalise `cond` to `0`/`1` with `cond != 0`, then turn it into an all-zero or all-ones mask so you can blend `a` and `b` bit for bit.
hint: With `m = -(cond != 0)` (`0` or `-1`), `(a & m) | (b & ~m)` keeps `a` when `m` is all-ones and `b` when `m` is all-zeros.
hint: The XOR form needs one fewer op: `b ^ ((a ^ b) & m)` — it is `b` when `m == 0` and `a` when `m == -1`.

```cpp
// starter
int bselect(int cond, int a, int b);
```

```cpp
int bselect(int cond, int a, int b) {
    int m = -(cond != 0);            // 0 or -1 (all bits set)
    return b ^ ((a ^ b) & m);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int cond, a, b, want; } cases[] = {
        {1, 10, 20, 10}, {0, 10, 20, 20}, {5, -1, INT_MIN, -1},
        {0, INT_MAX, INT_MIN, INT_MIN}, {1, INT_MIN, INT_MAX, INT_MIN},
        {-3, 7, 8, 7}, {0, 0, -1, -1}, {1, 0, -1, 0},
    };
    for (auto& c : cases) {
        int got = bselect(c.cond, c.a, c.b);
        if (got != c.want) { std::printf("bselect(%d,%d,%d)=%d want %d\n", c.cond, c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `cond != 0` collapses any truthy value to `1`; negating gives a mask `m` that is `0` or `-1` (all-ones). `b ^ ((a ^ b) & m)` reduces to `b` when `m == 0` and to `b ^ (a ^ b) == a` when `m == -1`, selecting without a branch. The equivalent `(a & m) | (b & ~m)` is the blend form. Because only bitwise operations touch `a` and `b`, there is no overflow even at `INT_MIN`. The payoff is on the hot path: an irregular `cond` would mispredict roughly half the time, each flush burning ~15-20 cycles, whereas the mask select is a fixed short chain of dependent instructions — exactly what a compiler produces as a `cmov` for `cond ? a : b`. O(1).
