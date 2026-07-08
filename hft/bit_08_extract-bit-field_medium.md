## challenge: Extract a bit field (offset + width)
tags: bit-tricks, hft
track: hft
difficulty: medium

Given a 32-bit unsigned integer and a field described by a starting bit `offset` and a `width`, return the `width` bits beginning at `offset`, right-aligned (shifted down to bit 0). Implement `uint32_t extractBits(uint32_t x, int offset, int width)`. Packed exchange messages and hardware registers cram several fields into one word; this is how you read one out.

Constraints: `0 <= offset <= 31`, `0 <= width <= 32`, `offset + width <= 32`. A `width` of `0` returns `0`. Beware the undefined behavior of shifting a 32-bit value by 32.

Example: `extractBits(0xDA, 1, 3) == 5` (0b11011010, bits 1..3 are 0b101). Example: `extractBits(0xFFFFFFFF, 4, 4) == 0xF`.

hint: Shift the field down by `offset` so it starts at bit 0, then mask off everything above `width`.
hint: The low-`width` mask is `(1u << width) - 1`, but `1u << 32` is undefined — special-case `width == 32`.
hint: Return `(x >> offset) & mask`.

```cpp
// starter
#include <cstdint>
uint32_t extractBits(uint32_t x, int offset, int width);
```

```cpp
uint32_t extractBits(uint32_t x, int offset, int width) {
    if (width == 0) return 0u;
    uint32_t mask = (width >= 32) ? 0xFFFFFFFFu : ((1u << width) - 1u);
    return (x >> offset) & mask;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x; int off, w; uint32_t want; } cases[] = {
        {0xDAu, 1, 3, 5u},                    // 0b11011010 -> bits 1..3 = 0b101
        {0xFFFFFFFFu, 4, 4, 0xFu},            // a nibble of ones
        {0xFFFFFFFFu, 0, 32, 0xFFFFFFFFu},    // full width (no UB shift)
        {0x80000000u, 31, 1, 1u},             // top bit alone
        {0u, 0, 8, 0u},                       // zero source
        {12345u, 3, 0, 0u},                   // zero width -> 0
    };
    for (auto& c : cases) {
        uint32_t got = extractBits(c.x, c.off, c.w);
        if (got != c.want) { std::printf("extractBits(%u,%d,%d)=%u want %u\n", c.x, c.off, c.w, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** A field is isolated in two moves: shift right by `offset` to bring its low bit to position 0, then AND with a mask of `width` ones to drop everything above it. The mask `(1u << width) - 1` produces `width` low ones — but shifting a 32-bit type by 32 is undefined in C++, so `width == 32` is handled explicitly as the all-ones mask, and `width == 0` returns 0 directly. Constant-time, branchless apart from those two boundary guards; hardware `BEXTR` performs the same extraction in one instruction.
