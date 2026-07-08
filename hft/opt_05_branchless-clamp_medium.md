## challenge: Branchless clamp to [lo, hi]
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

Clamping a value into a range shows up everywhere — bounding an order price, saturating an index. The obvious `if (x < lo) x = lo; else if (x > hi) x = hi;` is two data-dependent branches, each risking a ~15-20-cycle misprediction. Implement `int clampi(int x, int lo, int hi)` returning `x` clamped to `[lo, hi]` using two branchless selects (a branchless `max` then a branchless `min`).

Constraints: `lo <= hi`. All of `x`, `lo`, `hi` are any 32-bit `int` including `INT_MIN`/`INT_MAX`. Use only comparisons and bitwise selection — no additions, so no overflow.

Example: `clampi(5, 0, 10)` → `5`. Example: `clampi(-3, 0, 10)` → `0`. Example: `clampi(99, 0, 10)` → `10`. Example: `clampi(INT_MAX, INT_MIN, 0)` → `0`.

hint: `clamp(x, lo, hi) == min(max(x, lo), hi)`; build each of `max` and `min` from the mask trick so neither step branches.
hint: `max(x, lo) = x ^ ((x ^ lo) & -(x < lo))` — it becomes `lo` when `x < lo` and `x` otherwise.
hint: Feed that result into `min(m, hi) = hi ^ ((m ^ hi) & -(m < hi))` to cap it at `hi`.

```cpp
// starter
int clampi(int x, int lo, int hi);
```

```cpp
int clampi(int x, int lo, int hi) {
    int m = x ^ ((x ^ lo) & -(x < lo));    // max(x, lo)
    return hi ^ ((m ^ hi) & -(m < hi));    // min(m, hi)
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int x, lo, hi, want; } cases[] = {
        {5, 0, 10, 5}, {-3, 0, 10, 0}, {99, 0, 10, 10}, {0, 0, 10, 0}, {10, 0, 10, 10},
        {INT_MAX, INT_MIN, 0, 0}, {INT_MIN, 0, INT_MAX, 0}, {INT_MIN, INT_MIN, INT_MAX, INT_MIN},
        {INT_MAX, INT_MIN, INT_MAX, INT_MAX}, {-7, -5, 5, -5},
    };
    for (auto& c : cases) {
        int got = clampi(c.x, c.lo, c.hi);
        if (got != c.want) { std::printf("clampi(%d,%d,%d)=%d want %d\n", c.x, c.lo, c.hi, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Clamping is a composition of `max` and `min`, and each is a masked select. `max(x, lo)` raises anything below `lo` up to `lo`; feeding the result into `min(·, hi)` caps anything above `hi`. Every step uses `-(cond)` to build an all-zeros/all-ones mask and an XOR to blend, so no conditional jump is emitted — the two `if`s that a naive clamp compiles to would each mispredict on irregular data for ~15-20 cycles. Only comparisons and bitwise ops touch the values, so even `INT_MIN`/`INT_MAX` inputs are safe. At `-O2` the compiler typically produces two `cmov`s for the same effect. O(1).
