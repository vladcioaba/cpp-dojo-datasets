## challenge: Isolate the lowest set bit
tags: bit-tricks, hot-path
track: hft
difficulty: easy

Given a 32-bit unsigned integer, return a value with only its lowest (least-significant) set bit kept and every other bit cleared. If the input is `0`, return `0`. Implement `uint32_t lowestSetBit(uint32_t x)`. This shows up whenever you walk a bitmask of ready events one at a time.

Constraints: `0 <= x <= 2^32 - 1`. Must run in O(1) with no loop.

Example: `lowestSetBit(0b1100) == 0b0100` (12 → 4). Example: `lowestSetBit(0xFFFFFFFF) == 1`.

hint: Two's complement negation flips every bit strictly above the lowest set bit while leaving that bit and the trailing zeros alone.
hint: `-x` equals `~x + 1`; ANDing `x` with it keeps exactly one bit — the lowest one.
hint: Return `x & -x` (equivalently `x & (~x + 1)`).

```cpp
// starter
#include <cstdint>
uint32_t lowestSetBit(uint32_t x);
```

```cpp
uint32_t lowestSetBit(uint32_t x) {
    return x & -x;   // -x == ~x + 1 in two's complement
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x, want; } cases[] = {
        {0u, 0u},                    // no bits set
        {1u, 1u},                    // already isolated
        {0b1100u, 0b0100u},          // 12 -> 4
        {0xFFFFFFFFu, 1u},           // all ones -> lowest bit
        {0x80000000u, 0x80000000u},  // INT_MIN pattern -> itself
        {16u, 16u},                  // power of two -> itself
    };
    for (auto& c : cases) {
        uint32_t got = lowestSetBit(c.x);
        if (got != c.want) { std::printf("lowestSetBit(%u)=%u want %u\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** In two's complement, `-x = ~x + 1`. Negating flips every bit, then adding 1 ripples a carry up to and including the lowest set bit — so above that bit `x` and `-x` are exact complements (AND to 0), the lowest set bit is 1 in both (AND to 1), and the trailing zeros stay 0. Thus `x & -x` isolates precisely the lowest set bit. Constant-time, branchless — two ALU ops, no data-dependent jumps.
