## challenge: Branchless saturating subtract
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: hard

Saturating subtract clamps `a - b` to `[INT_MIN, INT_MAX]` instead of overflowing. As with saturating add, the challenge is detecting signed overflow without invoking it (so no signed `a - b`) and doing it branchlessly so the ~15-20-cycle misprediction penalty never appears. Implement `int sat_sub(int a, int b)`.

Constraints: `a`, `b` any 32-bit `int` including `INT_MIN`/`INT_MAX`. Do the difference and the overflow test in `unsigned`. Positive overflow (e.g. `INT_MAX - (-1)`) saturates to `INT_MAX`; negative overflow (e.g. `INT_MIN - 1`) saturates to `INT_MIN`.

Example: `sat_sub(INT_MAX, -1)` → `INT_MAX`. Example: `sat_sub(INT_MIN, 1)` → `INT_MIN`. Example: `sat_sub(5, 3)` → `2`. Example: `sat_sub(3, 5)` → `-2`.

hint: Subtraction overflows only when `a` and `b` have *different* signs and the wrapped result's sign differs from `a`'s; note the sign-difference test is the opposite of the add case.
hint: `(ua ^ ub)` has bit 31 set when the inputs differ in sign; `(ua ^ diff)` has bit 31 set when the result's sign differs from `a`; AND them for the overflow bit.
hint: The limit is still `lim = (unsigned)INT_MAX + (ua >> 31)` (`INT_MAX` if `a >= 0`, else `INT_MIN`); arithmetic-shift the overflow bit to a mask and select.

```cpp
// starter
int sat_sub(int a, int b);
```

```cpp
int sat_sub(int a, int b) {
    unsigned ua   = (unsigned)a;
    unsigned ub   = (unsigned)b;
    unsigned diff = ua - ub;                             // wraps, no UB
    unsigned lim  = (unsigned)INT_MAX + (ua >> 31);      // INT_MAX or INT_MIN
    // overflow iff a,b differ in sign (ua^ub sign bit) AND diff sign differs from a (ua^diff sign bit)
    unsigned ovf  = (unsigned)(((int)((ua ^ ub) & (ua ^ diff))) >> 31);
    return (int)((diff & ~ovf) | (lim & ovf));
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b, want; } cases[] = {
        {INT_MAX, -1, INT_MAX}, {INT_MIN, 1, INT_MIN}, {5, 3, 2}, {3, 5, -2},
        {INT_MIN, INT_MAX, INT_MIN}, {INT_MAX, INT_MIN, INT_MAX},
        {0, INT_MIN, INT_MAX}, {INT_MIN, INT_MIN, 0}, {INT_MAX, INT_MAX, 0},
        {-2000000000, 2000000000, INT_MIN}, {2000000000, -2000000000, INT_MAX}, {0, 0, 0},
    };
    for (auto& c : cases) {
        int got = sat_sub(c.a, c.b);
        if (got != c.want) { std::printf("sat_sub(%d,%d)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Because signed overflow is UB, the difference `diff = ua - ub` is taken in `unsigned`, where it wraps deterministically. Subtraction can overflow only when the operands differ in sign and the result's sign ends up differing from `a`'s — the mirror of the add rule. `ua ^ ub` has bit 31 set when the signs differ, `ua ^ diff` has bit 31 set when the result's sign flipped relative to `a`, and their AND isolates the overflow bit; an arithmetic right shift by 31 turns it into an all-ones/all-zeros mask. The saturation target `(unsigned)INT_MAX + (ua >> 31)` is `INT_MAX` for `a >= 0` and `INT_MIN` for `a < 0`, matching the direction of the overflow. `(diff & ~ovf) | (lim & ovf)` selects branchlessly, so an irregular overflow pattern never triggers a ~15-20-cycle pipeline flush, and every step is defined under C++20's two's-complement rules. O(1).
