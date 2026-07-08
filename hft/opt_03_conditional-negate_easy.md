## challenge: Conditional negate
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: easy

You often need "negate this value only if a flag is set" — e.g. applying a sign bit decoded from a message. Written as `flag ? -x : x` it is a branch on runtime data, mispredicting for ~15-20 cycles when the flag pattern is irregular. Implement `int cneg(int x, int flag)` that returns `-x` when `flag` is nonzero and `x` when `flag` is `0`, with no branch and no undefined behaviour.

Constraints: `x` is any 32-bit `int` including `INT_MIN`; `flag` is any int treated as a boolean. Negating `INT_MIN` is not representable, so `cneg(INT_MIN, 1)` must return `INT_MIN` (the two's-complement wraparound result) without triggering signed-overflow UB — do the arithmetic in `unsigned`.

Example: `cneg(5, 1)` → `-5`. Example: `cneg(5, 0)` → `5`. Example: `cneg(-3, 1)` → `3`. Example: `cneg(INT_MIN, 1)` → `INT_MIN`.

hint: Two's-complement negation is `~x + 1`, i.e. `(x ^ -1) + 1`; generalise the `-1` into a mask that is either all-ones (negate) or all-zeros (leave alone).
hint: With mask `m` all-ones or all-zeros, `(x ^ m) - m` equals `-x` when `m == -1` and `x` when `m == 0`.
hint: Build `m` from the flag in `unsigned` so the wrap on `INT_MIN` is well-defined: `unsigned m = -(unsigned)(flag != 0);`.

```cpp
// starter
int cneg(int x, int flag);
```

```cpp
int cneg(int x, int flag) {
    unsigned m = -(unsigned)(flag != 0);   // 0x00000000 or 0xFFFFFFFF
    return (int)(((unsigned)x ^ m) - m);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int x, flag, want; } cases[] = {
        {5, 1, -5}, {5, 0, 5}, {-3, 1, 3}, {-3, 0, -3},
        {0, 1, 0}, {0, 0, 0}, {INT_MAX, 1, -INT_MAX}, {INT_MIN, 1, INT_MIN},
        {INT_MIN, 0, INT_MIN}, {7, 42, -7},
    };
    for (auto& c : cases) {
        int got = cneg(c.x, c.flag);
        if (got != c.want) { std::printf("cneg(%d,%d)=%d want %d\n", c.x, c.flag, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Two's-complement negation is `~x + 1`. Replace the constant `~` (i.e. XOR with `-1`) by a mask `m` that is all-ones when you want to negate and all-zeros otherwise: `(x ^ m) - m` gives `-x` for `m == 0xFFFFFFFF` and `x` for `m == 0`. Building `m` and doing the subtract in `unsigned` keeps everything well-defined — critically, `cneg(INT_MIN, 1)` wraps to `INT_MIN` instead of the UB that `-x` would invoke, and in C++20 the final narrowing conversion back to `int` is defined to wrap. There is no branch, so the irregular flag stream never triggers a ~15-20-cycle misprediction; the compiler emits an XOR, a subtract, and a mask setup instead of a conditional jump. O(1).
