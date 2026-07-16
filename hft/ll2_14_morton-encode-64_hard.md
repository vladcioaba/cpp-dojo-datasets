## challenge: Morton encode two 32-bit coordinates
tags: bit-tricks, hot-path
track: hft
difficulty: hard

Z-order (Morton) curves map 2D coordinates to a single index so that points close in 2D stay close in memory — used for spatial grids, quadtrees, and cache-friendly 2D tables. Implement `uint64_t mortonEncode(uint32_t x, uint32_t y)` that interleaves the bits: bit `i` of `x` goes to bit `2i` of the result (even positions), bit `i` of `y` goes to bit `2i+1` (odd positions). No per-bit loops: use the parallel-prefix mask ladder that spreads each 32-bit input into 64 bits in 5 fixed steps.

Constraints: full 32-bit coordinate range; O(1) — a fixed sequence of shifts, ORs, and ANDs with the spreading mask constants; no loops, no tables, no builtins.

Example: `mortonEncode(1, 0) == 1`, `mortonEncode(0, 1) == 2`, `mortonEncode(3, 0) == 5` (binary `101`), `mortonEncode(0, 3) == 10` (binary `1010`), `mortonEncode(0xFFFFFFFF, 0) == 0x5555555555555555`.

hint: Write a helper that spreads one 32-bit value so its bits occupy the even positions of a 64-bit word, then combine: `spread(x) | (spread(y) << 1)`.
hint: The spread halves the gap each step with magic masks: `v = (v | (v << 16)) & 0x0000FFFF0000FFFF; v = (v | (v << 8)) & 0x00FF00FF00FF00FF;` then masks `0x0F0F...`, `0x3333...`, `0x5555...`.
hint: Each step moves the upper half of every block 2x its current gap to the left and the mask keeps exactly one half-block at each position — 16, 8, 4, 2, 1.

```cpp
// starter
#include <cstdint>
uint64_t mortonEncode(uint32_t x, uint32_t y);
```

```cpp
static uint64_t spread(uint64_t v) {
    // Spread the low 32 bits of v to the even bit positions of a 64-bit word.
    v &= 0x00000000FFFFFFFFull;
    v = (v | (v << 16)) & 0x0000FFFF0000FFFFull;
    v = (v | (v << 8))  & 0x00FF00FF00FF00FFull;
    v = (v | (v << 4))  & 0x0F0F0F0F0F0F0F0Full;
    v = (v | (v << 2))  & 0x3333333333333333ull;
    v = (v | (v << 1))  & 0x5555555555555555ull;
    return v;
}

uint64_t mortonEncode(uint32_t x, uint32_t y) {
    return spread(x) | (spread(y) << 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static uint64_t refEncode(uint32_t x, uint32_t y) {
    uint64_t r = 0;
    for (int i = 0; i < 32; ++i) {
        r |= (uint64_t)((x >> i) & 1u) << (2 * i);
        r |= (uint64_t)((y >> i) & 1u) << (2 * i + 1);
    }
    return r;
}
int main() {
    struct { uint32_t x, y; uint64_t want; } fixed[] = {
        {0u, 0u, 0ull},
        {1u, 0u, 1ull},
        {0u, 1u, 2ull},
        {3u, 0u, 5ull},
        {0u, 3u, 10ull},
        {0xFFFFFFFFu, 0u, 0x5555555555555555ull},
        {0u, 0xFFFFFFFFu, 0xAAAAAAAAAAAAAAAAull},
        {0xFFFFFFFFu, 0xFFFFFFFFu, 0xFFFFFFFFFFFFFFFFull},
    };
    for (auto& c : fixed) {
        uint64_t got = mortonEncode(c.x, c.y);
        if (got != c.want) {
            std::printf("mortonEncode(%u,%u)=%llx want %llx\n",
                        c.x, c.y, (unsigned long long)got, (unsigned long long)c.want);
            return 1;
        }
    }
    struct { uint32_t x, y; } pairs[] = {
        {0x12345678u, 0x9ABCDEF0u}, {0xDEADBEEFu, 0xCAFEBABEu},
        {0x80000000u, 0x00000001u}, {0x00000001u, 0x80000000u}, {12345u, 67890u},
    };
    for (auto& p : pairs) {
        uint64_t got = mortonEncode(p.x, p.y);
        uint64_t want = refEncode(p.x, p.y);
        if (got != want) {
            std::printf("mortonEncode(%x,%x)=%llx want %llx\n",
                        p.x, p.y, (unsigned long long)got, (unsigned long long)want);
            return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** The spread is a parallel-prefix computation run in reverse: start with 32 bits packed at the bottom of a 64-bit word and, in each step, split every contiguous block in half and move the upper half left by the block's width — 16, then 8, 4, 2, 1 — so after five steps each original bit sits alone at an even position. The magic masks are the invariant keepers: after the shift-and-OR, each value appears twice (original and shifted); the mask (`0x0000FFFF0000FFFF`, then `0x00FF00FF...`, `0x0F0F...`, `0x3333...`, `0x5555...`) keeps exactly the copy that belongs at each position. Interleaving is then trivial: `x` on the even bits, `y` shifted onto the odd bits, OR them — the two spreads are independent, so a superscalar core overlaps them. Why Morton order matters for latency: a row-major 2D grid puts vertical neighbors a full row apart (guaranteed cache miss for large grids), while Z-order keeps any 2^k x 2^k tile in one contiguous memory range — that locality is why GPUs swizzle textures this way and why spatial indexes (and some order-book-by-(price, venue) grids) use it. On x86 with BMI2 the whole spread is one `pdep` instruction; this ladder is its portable, constexpr-able equivalent, and running it backwards (mask, then compact with shifts) gives you the decode.
