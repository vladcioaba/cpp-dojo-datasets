## challenge: Gray code encode and decode
tags: bit-tricks, hft
track: hft
difficulty: hard

Implement the reflected binary (Gray) code round trip on 32-bit unsigned integers: `uint32_t grayEncode(uint32_t x)` maps a binary value to its Gray code (consecutive values differ in exactly one bit), and `uint32_t grayDecode(uint32_t g)` inverts it. `grayDecode(grayEncode(x)) == x` for every `x`. Gray coding lets a rotary/optical encoder cross a boundary without transient glitches, and appears in clock-domain-crossing FIFO pointers.

Constraints: `0 <= x, g <= 2^32 - 1`. Encode is O(1); decode is O(log bits). No loop over individual values.

Example: `grayEncode(2) == 3`, `grayEncode(3) == 2`, `grayEncode(4) == 6`. Example: `grayDecode(6) == 4`.

hint: Encoding is a single XOR of the value with itself shifted right by one — each Gray bit is the XOR of two adjacent binary bits.
hint: Decoding must undo a prefix-XOR, so it XOR-folds the shifted copies back together (shifts of 16, 8, 4, 2, 1).
hint: `encode: x ^ (x >> 1)`; `decode: g ^= g>>16; g^=g>>8; ... g^=g>>1;`.

```cpp
// starter
#include <cstdint>
uint32_t grayEncode(uint32_t x);
uint32_t grayDecode(uint32_t g);
```

```cpp
uint32_t grayEncode(uint32_t x) {
    return x ^ (x >> 1);
}

uint32_t grayDecode(uint32_t g) {
    g ^= g >> 16;
    g ^= g >> 8;
    g ^= g >> 4;
    g ^= g >> 2;
    g ^= g >> 1;
    return g;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    // Known encodings: consecutive values differ in exactly one bit.
    struct { uint32_t x, want; } enc[] = {
        {0u, 0u}, {1u, 1u}, {2u, 3u}, {3u, 2u}, {4u, 6u},
        {0xFFFFFFFFu, 0x80000000u},   // all ones -> single high bit
    };
    for (auto& c : enc) {
        uint32_t got = grayEncode(c.x);
        if (got != c.want) { std::printf("grayEncode(%u)=0x%08X want 0x%08X\n", c.x, got, c.want); return 1; }
    }
    // Round-trip decode(encode(x)) == x, including edge patterns.
    uint32_t probes[] = {0u, 1u, 2u, 4u, 255u, 0x80000000u, 0xFFFFFFFFu, 0xDEADBEEFu};
    for (uint32_t x : probes) {
        uint32_t back = grayDecode(grayEncode(x));
        if (back != x) { std::printf("roundtrip x=0x%08X got 0x%08X\n", x, back); return 1; }
    }
    // Adjacent Gray codes must differ in exactly one bit.
    for (uint32_t x = 0; x < 64; ++x) {
        uint32_t d = grayEncode(x) ^ grayEncode(x + 1);
        if ((d & (d - 1)) != 0u || d == 0u) { std::printf("not single-bit at x=%u\n", x); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Gray bit `i` is defined as binary bit `i` XOR binary bit `i+1`, which in one shot is `x ^ (x >> 1)` — so incrementing the binary value flips exactly one Gray bit (the encoder never shows a half-updated code). Decoding must recover binary bit `i` as the XOR of *all* Gray bits from `i` up to the top (a suffix parity). Doing that naively is a 31-step chain `b[i] = g[i] ^ b[i+1]`; the shift-fold `g ^= g>>16; g^=g>>8; ...; g^=g>>1` computes the same suffix-XOR for all bits in parallel in log steps. Encode is one XOR (O(1)); decode is five XOR-shift pairs (O(log 32)). Both are branchless — no data-dependent control flow.
