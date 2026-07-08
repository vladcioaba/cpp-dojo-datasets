## challenge: Compute parity of a word
tags: bit-tricks, hft
track: hft
difficulty: medium

Given a 64-bit unsigned integer, return `1` if it has an odd number of set bits and `0` if even — its parity. Implement `int parity(uint64_t x)` without `__builtin_parityll`/`std::popcount`. Parity is the 1-bit checksum behind RAID, ECC, and many exchange line protocols.

Constraints: `0 <= x <= 2^64 - 1`. O(log bits) — no per-bit loop.

Example: `parity(7) == 1` (three set bits). Example: `parity(3) == 0` (two set bits), `parity(0) == 0`.

hint: Parity is the XOR of all bits. XOR is associative, so you can fold the word in halves instead of scanning bit by bit.
hint: `x ^= x >> 32; x ^= x >> 16; ... x ^= x >> 1;` collapses the parity of all 64 bits into bit 0.
hint: Return the final low bit: `x & 1`.

```cpp
// starter
#include <cstdint>
int parity(uint64_t x);
```

```cpp
int parity(uint64_t x) {
    x ^= x >> 32;
    x ^= x >> 16;
    x ^= x >> 8;
    x ^= x >> 4;
    x ^= x >> 2;
    x ^= x >> 1;
    return (int)(x & 1u);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x; int want; } cases[] = {
        {0ull, 0},                       // no bits
        {1ull, 1},                       // one bit -> odd
        {3ull, 0},                       // two bits -> even
        {7ull, 1},                       // three bits -> odd
        {0x8000000000000000ull, 1},      // single high bit
        {0xFFFFFFFFFFFFFFFFull, 0},      // 64 bits -> even
        {0xAAAAAAAAull, 0},              // 16 bits -> even
    };
    for (auto& c : cases) {
        int got = parity(c.x);
        if (got != c.want) { std::printf("parity(%llu)=%d want %d\n", (unsigned long long)c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Parity is the XOR reduction of all bits, and XOR is associative and commutative, so a tree fold is legal. Each step `x ^= x >> k` (with k = 32, 16, 8, 4, 2, 1) XORs the upper half of the still-live window into the lower half; after the last step, bit 0 holds the XOR of all 64 original bits. Masking `x & 1` reads it off. Six shift/XOR pairs — O(log 64) = constant work, fully branchless. (On x86, `popcnt` plus `& 1` is the one-instruction alternative.)
