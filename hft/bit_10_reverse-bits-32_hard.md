## challenge: Reverse the bits of a 32-bit word
tags: bit-tricks, hft
track: hft
difficulty: hard

Given a 32-bit unsigned integer, return the value with its bit order reversed: bit 0 becomes bit 31, bit 1 becomes bit 30, and so on. Implement `uint32_t reverseBits(uint32_t x)` in O(log bits) using a parallel swap network — no per-bit loop. Bit-reversal permutations appear in FFTs and in some hardware address remaps.

Constraints: `0 <= x <= 2^32 - 1`. Fixed number of steps regardless of input.

Example: `reverseBits(1) == 0x80000000`. Example: `reverseBits(0xFFFF0000) == 0x0000FFFF`.

hint: Reverse by repeatedly swapping ever-larger blocks: first swap adjacent bits, then 2-bit groups, then nibbles, bytes, and finally the two halves.
hint: Each stage is a masked shift pair, e.g. swap odd/even bits with masks `0xAAAAAAAA` and `0x55555555`.
hint: Five stages (1, 2, 4, 8, 16-bit swaps) fully reverse a 32-bit word.

```cpp
// starter
#include <cstdint>
uint32_t reverseBits(uint32_t x);
```

```cpp
uint32_t reverseBits(uint32_t x) {
    x = ((x & 0xAAAAAAAAu) >> 1)  | ((x & 0x55555555u) << 1);   // swap adjacent bits
    x = ((x & 0xCCCCCCCCu) >> 2)  | ((x & 0x33333333u) << 2);   // swap 2-bit groups
    x = ((x & 0xF0F0F0F0u) >> 4)  | ((x & 0x0F0F0F0Fu) << 4);   // swap nibbles
    x = ((x & 0xFF00FF00u) >> 8)  | ((x & 0x00FF00FFu) << 8);   // swap bytes
    x = (x >> 16) | (x << 16);                                  // swap halves
    return x;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x, want; } cases[] = {
        {0u, 0u},                          // symmetric
        {0xFFFFFFFFu, 0xFFFFFFFFu},        // all ones symmetric
        {1u, 0x80000000u},                 // bit 0 -> bit 31
        {0x80000000u, 1u},                 // bit 31 -> bit 0
        {0x00000002u, 0x40000000u},        // bit 1 -> bit 30
        {0xFFFF0000u, 0x0000FFFFu},        // upper half -> lower half
        {0xAAAAAAAAu, 0x55555555u},        // even bits -> odd bits
    };
    for (auto& c : cases) {
        uint32_t got = reverseBits(c.x);
        if (got != c.want) { std::printf("reverseBits(0x%08X)=0x%08X want 0x%08X\n", c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** A full bit-reversal is a sequence of swaps at doubling granularity. Stage 1 exchanges each even/odd bit pair (mask the odd bits down, the even bits up, OR them). Stage 2 swaps 2-bit groups, stage 3 nibbles, stage 4 bytes, stage 5 the two 16-bit halves. After all five, the bit that started at position `i` ends at `31 - i`. This is O(log 32) = 5 stages of constant work — fully branchless, no data-dependent shift counts, and far faster than a 32-iteration shift-and-test loop. Some ISAs expose it directly (ARM `RBIT`).
