## challenge: Turn off the trailing run of 1s
tags: bit-tricks, hot-path
track: hft
difficulty: medium

Given a 32-bit unsigned integer, clear the contiguous run of 1-bits at its least-significant end, leaving every bit above that run unchanged. If the lowest bit is already 0 (no trailing run), return the value untouched. Implement `uint32_t turnOffTrailingOnes(uint32_t x)`. It is the dual of clearing the lowest set bit and shows up when releasing a block of consecutively allocated slots.

Constraints: `0 <= x <= 2^32 - 1`. O(1), no loop.

Example: `turnOffTrailingOnes(0b1011) == 0b1000` (11 → 8). Example: `turnOffTrailingOnes(0b10110111) == 0b10110000` (183 → 176).

hint: Adding 1 propagates a carry through exactly the trailing run of 1s, flipping them to 0 and setting the next 0 to 1.
hint: ANDing that result with the original keeps the high bits and wipes the trailing ones that got flipped.
hint: Return `x & (x + 1)`.

```cpp
// starter
#include <cstdint>
uint32_t turnOffTrailingOnes(uint32_t x);
```

```cpp
uint32_t turnOffTrailingOnes(uint32_t x) {
    return x & (x + 1u);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x, want; } cases[] = {
        {0u, 0u},                    // no trailing ones -> unchanged
        {7u, 0u},                    // 0b111 -> all cleared
        {0b1011u, 0b1000u},          // 11 -> 8
        {0b10110111u, 0b10110000u},  // 183 -> 176
        {8u, 8u},                    // power of two: bit0 is 0 -> unchanged
        {0xFFFFFFFFu, 0u},           // all ones -> everything cleared
        {0x80000000u, 0x80000000u},  // top bit, no trailing run -> unchanged
    };
    for (auto& c : cases) {
        uint32_t got = turnOffTrailingOnes(c.x);
        if (got != c.want) { std::printf("turnOffTrailingOnes(%u)=%u want %u\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `x + 1` adds a carry that ripples through the trailing run of 1s, turning each into 0 and setting the first higher 0-bit to 1; bits above that are untouched. ANDing with the original keeps the unchanged high bits, and the trailing positions — now 1 in `x` but 0 in `x + 1` — clear to 0. If the lowest bit is already 0 there is no run: `x + 1` only sets bit 0, and `x & (x + 1) == x`, leaving the value alone. For all-ones the carry runs off the top (wraps to 0) and everything clears. Constant-time, branchless — the mirror image of `x & (x - 1)`.
