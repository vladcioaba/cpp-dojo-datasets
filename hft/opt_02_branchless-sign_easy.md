## challenge: Branchless sign of an int
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: easy

The sign function returns `-1` for negatives, `0` for zero, and `+1` for positives. Written with `if`s it is three data-dependent branches on the hot path; each unpredictable one can cost ~15-20 cycles when it mispredicts. Implement `int sign(int x)` with no branches by subtracting two boolean comparisons.

Constraints: `x` is any 32-bit `int`, including `INT_MIN` and `INT_MAX`. Do not compute `-x` or `x` in a way that could overflow — comparisons against `0` are enough.

Example: `sign(42)` → `1`. Example: `sign(-7)` → `-1`. Example: `sign(0)` → `0`. Example: `sign(INT_MIN)` → `-1`.

hint: Two comparisons, each yielding `0` or `1`, can be combined so their difference lands in `{-1, 0, 1}` — no branch and no negation of `x` itself.
hint: `(x > 0)` is `1` exactly for positives; `(x < 0)` is `1` exactly for negatives; they are never both `1`.
hint: Subtract the "is-negative" flag from the "is-positive" flag: `(x > 0) - (x < 0)`.

```cpp
// starter
int sign(int x);
```

```cpp
int sign(int x) {
    return (x > 0) - (x < 0);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int x, want; } cases[] = {
        {42, 1}, {-7, -1}, {0, 0}, {1, 1}, {-1, -1},
        {INT_MAX, 1}, {INT_MIN, -1}, {2147483646, 1}, {-2147483647, -1},
    };
    for (auto& c : cases) {
        int got = sign(c.x);
        if (got != c.want) { std::printf("sign(%d)=%d want %d\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `(x > 0)` and `(x < 0)` each evaluate to `0` or `1` and are mutually exclusive, so their difference is exactly `+1`, `-1`, or `0`. No value is ever negated, so `INT_MIN` is safe — the naive `x < 0 ? -1 : (x > 0 ? 1 : 0)` is correct too but introduces two branches that a random sign stream mispredicts about half the time, each flush costing ~15-20 cycles. The subtraction form has none: a compiler lowers each comparison to a `setcc` (or a shift-based idiom) and emits one subtract, so the whole function is a handful of dependent, jump-free instructions. O(1), no overflow.
