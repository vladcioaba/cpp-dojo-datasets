## challenge: Count trailing zeros
tags: bit-tricks, hot-path
track: hft
difficulty: medium

Given a 32-bit unsigned integer, return the number of consecutive zero bits starting from the least-significant end (the position of the lowest set bit). Define `countTrailingZeros(0) == 32`. Implement `int countTrailingZeros(uint32_t x)` without using `__builtin_ctz`/`std::countr_zero`. This is how you convert an isolated ready-bit back into an index.

Constraints: `0 <= x <= 2^32 - 1`. Must be O(1) — a fixed number of steps, not a per-bit loop.

Example: `countTrailingZeros(0b1000) == 3`. Example: `countTrailingZeros(0x80000000) == 31`, `countTrailingZeros(1) == 0`.

hint: Binary-search the position: ask "are the low 16 bits all zero?", then the low 8 of what's left, and so on, accumulating the shift.
hint: Each test either adds a power-of-two count and shifts, or does nothing — five tests cover 32 bits.
hint: Handle `x == 0` up front (answer 32); otherwise the halving search lands exactly on the lowest set bit.

```cpp
// starter
#include <cstdint>
int countTrailingZeros(uint32_t x);
```

```cpp
int countTrailingZeros(uint32_t x) {
    if (x == 0u) return 32;
    int n = 0;
    if ((x & 0x0000FFFFu) == 0u) { n += 16; x >>= 16; }
    if ((x & 0x000000FFu) == 0u) { n += 8;  x >>= 8;  }
    if ((x & 0x0000000Fu) == 0u) { n += 4;  x >>= 4;  }
    if ((x & 0x00000003u) == 0u) { n += 2;  x >>= 2;  }
    if ((x & 0x00000001u) == 0u) { n += 1;             }
    return n;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x; int want; } cases[] = {
        {0u, 32},              // by definition
        {1u, 0},               // lowest bit already set
        {0b1000u, 3},          // 8
        {12u, 2},              // 0b1100 -> lowest set bit at 2
        {16u, 4},              // power of two
        {0x80000000u, 31},     // only the top bit
        {0xFFFFFFFFu, 0},      // all ones -> no trailing zeros
    };
    for (auto& c : cases) {
        int got = countTrailingZeros(c.x);
        if (got != c.want) { std::printf("countTrailingZeros(%u)=%d want %d\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The lowest set bit's index is found by binary search over the 32 positions. If the low 16 bits are all zero the answer is at least 16, so add 16 and shift the upper half down; repeat with 8, 4, 2, 1. Five conditional adds pin the exact position — total work is constant regardless of input. The `x == 0` guard supplies the conventional answer 32 (there is no set bit to point at). Each test is a mask-compare that most compilers lower to `cmov`, so the routine is effectively branchless; where available the hardware `tzcnt`/`bsf` instruction does the same in one op.
