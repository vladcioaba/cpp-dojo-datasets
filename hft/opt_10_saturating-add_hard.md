## challenge: Branchless saturating add
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: hard

Saturating add clamps `a + b` to `[INT_MIN, INT_MAX]` instead of overflowing — needed for fixed-point accumulators and counters that must not wrap. The tricky part is detecting signed overflow *without* triggering it (signed overflow is UB, so `a + b` is off-limits), and doing it with no branch so the ~15-20-cycle misprediction penalty never appears. Implement `int sat_add(int a, int b)`.

Constraints: `a`, `b` any 32-bit `int` including `INT_MIN`/`INT_MAX`. Compute the sum and the overflow test entirely in `unsigned`; positive overflow saturates to `INT_MAX`, negative overflow to `INT_MIN`.

Example: `sat_add(INT_MAX, 1)` → `INT_MAX`. Example: `sat_add(INT_MIN, -1)` → `INT_MIN`. Example: `sat_add(5, 3)` → `8`. Example: `sat_add(-5, 3)` → `-2`.

hint: Overflow happens only when `a` and `b` share a sign but the wrapped `sum` has the opposite sign; encode "same input sign" and "sum sign flipped" as bit-31 tests and AND them.
hint: The saturation limit is `INT_MAX` when `a >= 0` and `INT_MIN` when `a < 0`: `lim = (unsigned)INT_MAX + (ua >> 31)` gives `0x7FFFFFFF` or `0x80000000`.
hint: Turn the overflow bit into an all-ones/all-zeros mask with an arithmetic right shift by 31, then branchlessly select between `sum` and `lim`.

```cpp
// starter
int sat_add(int a, int b);
```

```cpp
int sat_add(int a, int b) {
    unsigned ua  = (unsigned)a;
    unsigned ub  = (unsigned)b;
    unsigned sum = ua + ub;                              // wraps, no UB
    unsigned lim = (unsigned)INT_MAX + (ua >> 31);       // INT_MAX or INT_MIN
    // overflow iff a,b same sign (~(ua^ub) sign bit) AND sum sign differs from a (ua^sum sign bit)
    unsigned ovf = (unsigned)(((int)(~(ua ^ ub) & (ua ^ sum))) >> 31);
    return (int)((sum & ~ovf) | (lim & ovf));
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b, want; } cases[] = {
        {INT_MAX, 1, INT_MAX}, {INT_MIN, -1, INT_MIN}, {5, 3, 8}, {-5, 3, -2},
        {INT_MAX, INT_MAX, INT_MAX}, {INT_MIN, INT_MIN, INT_MIN},
        {INT_MAX, INT_MIN, -1}, {2000000000, 2000000000, INT_MAX},
        {-2000000000, -2000000000, INT_MIN}, {0, 0, 0}, {INT_MAX, 0, INT_MAX}, {INT_MIN, 0, INT_MIN},
    };
    for (auto& c : cases) {
        int got = sat_add(c.a, c.b);
        if (got != c.want) { std::printf("sat_add(%d,%d)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Signed overflow is UB, so the whole computation lives in `unsigned`, where `sum = ua + ub` wraps deterministically. Overflow can only occur when the operands share a sign yet the result's sign flips: `~(ua ^ ub)` has bit 31 set exactly when `a` and `b` have the same sign, and `ua ^ sum` has bit 31 set exactly when the sum's sign differs from `a`'s — ANDing them isolates overflow in bit 31. An arithmetic right shift by 31 smears that bit into an all-ones/all-zeros mask `ovf`. The saturation target depends on direction: `(unsigned)INT_MAX + (ua >> 31)` is `0x7FFFFFFF` when `a >= 0` and `0x80000000` (`INT_MIN`) when `a < 0`. The final `(sum & ~ovf) | (lim & ovf)` is a branchless select. No conditional jump means the overflow path never costs a ~15-20-cycle misprediction, and C++20's defined signed-shift and unsigned-to-signed conversion keep it UB-free. O(1).
