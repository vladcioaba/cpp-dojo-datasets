## challenge: Bits to flip to convert A to B (Hamming distance)
tags: bit-tricks, hot-path
track: hft
difficulty: medium

Given two 32-bit unsigned integers, return how many bit positions differ — equivalently, the minimum number of single-bit flips to turn `a` into `b`. Implement `int hammingDistance(uint32_t a, uint32_t b)` without `__builtin_popcount`/`std::popcount`. It measures how far apart two bitmasks are, e.g. how many order-book levels changed between snapshots.

Constraints: `0 <= a, b <= 2^32 - 1`. O(1), no per-bit loop.

Example: `hammingDistance(1, 2) == 2` (bit 0 vs bit 1). Example: `hammingDistance(0xFFFFFFFF, 0) == 32`.

hint: The positions that differ are exactly the set bits of `a ^ b`; the answer is the population count of that XOR.
hint: Count bits in parallel with the SWAR trick: sum adjacent pairs, then nibbles, then bytes — no loop.
hint: The final `(v * 0x01010101) >> 24` sums the four per-byte counts into one number.

```cpp
// starter
#include <cstdint>
int hammingDistance(uint32_t a, uint32_t b);
```

```cpp
int hammingDistance(uint32_t a, uint32_t b) {
    uint32_t x = a ^ b;
    x = x - ((x >> 1) & 0x55555555u);
    x = (x & 0x33333333u) + ((x >> 2) & 0x33333333u);
    x = (x + (x >> 4)) & 0x0F0F0F0Fu;
    return (int)((x * 0x01010101u) >> 24);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t a, b; int want; } cases[] = {
        {0u, 0u, 0},                       // identical
        {1u, 0u, 1},                       // one bit differs
        {1u, 2u, 2},                       // 0b01 vs 0b10
        {0xFFFFFFFFu, 0u, 32},             // all bits differ
        {0xFFFFFFFFu, 0xFFFFFFFFu, 0},     // identical all-ones
        {0x80000000u, 0u, 1},              // top bit only
        {0xAAAAAAAAu, 0x55555555u, 32},    // fully complementary
    };
    for (auto& c : cases) {
        int got = hammingDistance(c.a, c.b);
        if (got != c.want) { std::printf("hammingDistance(%u,%u)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `a ^ b` has a 1 exactly where the inputs differ, so the Hamming distance is `popcount(a ^ b)`. The population count uses the classic SWAR (SIMD-within-a-register) fold: the first line replaces each 2-bit field with the count of set bits in it, the second sums those into 4-bit fields, the third into byte fields, and the multiply-by-`0x01010101` plus `>> 24` adds the four byte counts in one shot (the high byte of the product is their sum). No branches, no loop, a handful of ALU ops — O(1). On modern x86 this is what `popcnt(a ^ b)` does in hardware.
