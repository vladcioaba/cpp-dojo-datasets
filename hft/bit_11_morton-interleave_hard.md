## challenge: Interleave two 16-bit values (Morton code)
tags: bit-tricks, hft
track: hft
difficulty: hard

Given two 16-bit unsigned integers `x` and `y`, produce the 32-bit Morton code (Z-order value) that interleaves their bits: bit `i` of `x` lands in output position `2*i`, and bit `i` of `y` in position `2*i + 1`. Implement `uint32_t mortonInterleave(uint16_t x, uint16_t y)` with a parallel bit-spread — no per-bit loop. Morton codes linearize 2-D coordinates so nearby points stay near in memory.

Constraints: `0 <= x, y <= 65535`. Fixed number of steps.

Example: `mortonInterleave(1, 0) == 1`, `mortonInterleave(0, 1) == 2`, `mortonInterleave(1, 1) == 3`. Example: `mortonInterleave(0xFFFF, 0) == 0x55555555`.

hint: Spread each 16-bit value so its bits occupy only the even positions (gaps in between), then shift `y`'s spread left by one and OR.
hint: Spread by repeatedly splitting and shifting halves apart, masking with `0x00FF00FF`, `0x0F0F0F0F`, `0x33333333`, `0x55555555`.
hint: `part(x) | (part(y) << 1)` combines the two interleaved streams.

```cpp
// starter
#include <cstdint>
uint32_t mortonInterleave(uint16_t x, uint16_t y);
```

```cpp
static uint32_t spreadBits(uint32_t n) {
    n &= 0x0000FFFFu;
    n = (n | (n << 8)) & 0x00FF00FFu;
    n = (n | (n << 4)) & 0x0F0F0F0Fu;
    n = (n | (n << 2)) & 0x33333333u;
    n = (n | (n << 1)) & 0x55555555u;
    return n;
}

uint32_t mortonInterleave(uint16_t x, uint16_t y) {
    return spreadBits(x) | (spreadBits(y) << 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint16_t x, y; uint32_t want; } cases[] = {
        {0u, 0u, 0u},                       // origin
        {1u, 0u, 1u},                       // x bit0 -> pos0
        {0u, 1u, 2u},                       // y bit0 -> pos1
        {1u, 1u, 3u},                       // both bit0
        {0xFFFFu, 0u, 0x55555555u},         // x fills even positions
        {0u, 0xFFFfu, 0xAAAAAAAAu},         // y fills odd positions
        {0xFFFFu, 0xFFFFu, 0xFFFFFFFFu},    // fully packed
    };
    for (auto& c : cases) {
        uint32_t got = mortonInterleave(c.x, c.y);
        if (got != c.want) { std::printf("mortonInterleave(%u,%u)=0x%08X want 0x%08X\n", c.x, c.y, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `spreadBits` takes a 16-bit value and inserts a zero between each pair of adjacent bits, expanding 16 bits across 32 positions (bit `i` → position `2*i`). It does this in log steps: each `n = (n | (n << k)) & mask` splits the current groups in half and pushes them apart, with masks `0x00FF00FF`, `0x0F0F0F0F`, `0x33333333`, `0x55555555` keeping the bits in their new even slots. Spreading `x` fills the even positions; spreading `y` and shifting left by one fills the odd positions; OR-ing merges them. All constant work, no branches, no loop — O(1). (BMI2 `PDEP` with mask `0x55555555` does the spread in a single instruction.)
