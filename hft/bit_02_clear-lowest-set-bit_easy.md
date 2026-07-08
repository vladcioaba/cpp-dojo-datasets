## challenge: Clear the lowest set bit
tags: bit-tricks, hot-path
track: hft
difficulty: easy

Given a 32-bit unsigned integer, return it with its lowest (least-significant) set bit turned off; every other bit is unchanged. If the input is `0`, return `0`. Implement `uint32_t clearLowestSetBit(uint32_t x)`. This is the core step of the classic "iterate over set bits" loop and of Kernighan's popcount.

Constraints: `0 <= x <= 2^32 - 1`. O(1), no loop.

Example: `clearLowestSetBit(0b1100) == 0b1000` (12 → 8). Example: `clearLowestSetBit(7) == 6`.

hint: Subtracting 1 turns the lowest set bit into a 0 and all trailing zeros into 1s, leaving higher bits untouched.
hint: ANDing that result back with the original wipes exactly the lowest set bit.
hint: Return `x & (x - 1)`.

```cpp
// starter
#include <cstdint>
uint32_t clearLowestSetBit(uint32_t x);
```

```cpp
uint32_t clearLowestSetBit(uint32_t x) {
    return x & (x - 1u);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x, want; } cases[] = {
        {0u, 0u},                    // stays 0 (wraps but AND yields 0)
        {1u, 0u},                    // single bit cleared
        {0b1100u, 0b1000u},          // 12 -> 8
        {7u, 6u},                    // 0b111 -> 0b110
        {0xFFFFFFFFu, 0xFFFFFFFEu},  // all ones -> clears bit 0
        {0x80000000u, 0u},           // INT_MIN pattern -> only bit gone
    };
    for (auto& c : cases) {
        uint32_t got = clearLowestSetBit(c.x);
        if (got != c.want) { std::printf("clearLowestSetBit(%u)=%u want %u\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `x - 1` flips the lowest set bit to 0 and sets every bit below it to 1, while bits above are untouched by the borrow. ANDing with the original keeps the high bits, clears the (now-mismatched) lowest bit, and the trailing bits AND to 0. For `x == 0`, unsigned `x - 1` wraps to `0xFFFFFFFF`, and `0 & 0xFFFFFFFF == 0`, so the zero case is correct without a branch. Constant-time, branchless — this is why `while (x) x &= x - 1;` counts set bits in as many iterations as there are 1s.
