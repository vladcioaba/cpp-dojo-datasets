## challenge: Reverse the bits of a 64-bit word
tags: bit-tricks, hot-path
track: hft
difficulty: hard

Mirror a 64-bit word: bit 0 swaps with bit 63, bit 1 with bit 62, and so on. Implement `uint64_t reverseBits64(uint64_t v)` with the divide-and-conquer mask ladder — six fixed steps, no per-bit loop, no lookup tables, no builtins. Bit reversal is the index permutation at the heart of the FFT butterfly and appears in CRCs and LFSRs; the ladder technique itself (swap at doubling granularities) is a bit-manipulation staple interviewers reach for.

Constraints: full 64-bit range; O(1) — exactly six shift/mask/OR rounds (or five plus a final rotate); straight-line code.

Example: `reverseBits64(1) == 0x8000000000000000`, `reverseBits64(0x8000000000000000) == 1`, `reverseBits64(0xAAAAAAAAAAAAAAAA) == 0x5555555555555555`, `reverseBits64(0) == 0`.

hint: Swap adjacent bits, then adjacent 2-bit pairs, then nibbles, bytes, 16-bit halves, and finally the two 32-bit halves — reversing blocks at every scale composes into a full mirror.
hint: Each step is `v = ((v >> k) & M) | ((v & M) << k)` where `M` keeps the low half of every 2k-block: `0x5555...` for k=1, `0x3333...` for k=2, `0x0F0F...` for k=4, `0x00FF...` for k=8, `0x0000FFFF...` for k=16.
hint: The last step needs no mask at all: `v = (v >> 32) | (v << 32)` swaps the halves in one rotate.

```cpp
// starter
#include <cstdint>
uint64_t reverseBits64(uint64_t v);
```

```cpp
uint64_t reverseBits64(uint64_t v) {
    v = ((v >> 1)  & 0x5555555555555555ull) | ((v & 0x5555555555555555ull) << 1);   // swap bits
    v = ((v >> 2)  & 0x3333333333333333ull) | ((v & 0x3333333333333333ull) << 2);   // swap pairs
    v = ((v >> 4)  & 0x0F0F0F0F0F0F0F0Full) | ((v & 0x0F0F0F0F0F0F0F0Full) << 4);   // swap nibbles
    v = ((v >> 8)  & 0x00FF00FF00FF00FFull) | ((v & 0x00FF00FF00FF00FFull) << 8);   // swap bytes
    v = ((v >> 16) & 0x0000FFFF0000FFFFull) | ((v & 0x0000FFFF0000FFFFull) << 16);  // swap 16-bit
    v = (v >> 32) | (v << 32);                                                      // swap halves
    return v;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static uint64_t refReverse(uint64_t v) {
    uint64_t r = 0;
    for (int i = 0; i < 64; ++i) {
        r = (r << 1) | (v & 1u);
        v >>= 1;
    }
    return r;
}
int main() {
    struct { uint64_t v, want; } fixed[] = {
        {0ull, 0ull},
        {1ull, 0x8000000000000000ull},
        {0x8000000000000000ull, 1ull},
        {0xFFFFFFFFFFFFFFFFull, 0xFFFFFFFFFFFFFFFFull},
        {0xAAAAAAAAAAAAAAAAull, 0x5555555555555555ull},
        {0x5555555555555555ull, 0xAAAAAAAAAAAAAAAAull},
        {0x00000000000000FFull, 0xFF00000000000000ull},
    };
    for (auto& c : fixed) {
        uint64_t got = reverseBits64(c.v);
        if (got != c.want) {
            std::printf("reverseBits64(%llx)=%llx want %llx\n",
                        (unsigned long long)c.v, (unsigned long long)got, (unsigned long long)c.want);
            return 1;
        }
    }
    uint64_t vals[] = {0x0123456789ABCDEFull, 0xF0F0F0F0F0F0F0F0ull, 0xDEADBEEFCAFEBABEull, 42ull};
    for (uint64_t v : vals) {
        if (reverseBits64(v) != refReverse(v)) { std::printf("mismatch on %llx\n", (unsigned long long)v); return 1; }
        if (reverseBits64(reverseBits64(v)) != v) { std::puts("reverse must be an involution"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The key insight is that a full mirror factors into log2(64) = 6 independent block swaps: reversing a 64-bit string equals swapping its two 32-bit halves *and* reversing each half, and that recursion unrolls into "swap adjacent 1-bit blocks, then 2-bit, 4, 8, 16, 32" — in any order, since the steps commute. Each round is the two-mask exchange `((v >> k) & M) | ((v & M) << k)`: `M` selects the low half of every 2k-sized block, so the expression moves high halves down and low halves up simultaneously across the whole word — 64 swaps for the price of 5 ops. The final 32-bit step degenerates to a rotate because the mask would cover exactly half the word. Total: ~28 ALU ops, no branches, no memory — compare a naive 64-iteration loop (~200+ ops with a loop-carried dependency) or a 256-entry byte table (4 cache lines of L1 you'd rather spend on the order book). ARM has `rbit` doing this in one instruction; x86 doesn't, so the ladder is what fast FFT index permutation actually compiles to. The same exchange idiom generalizes: byte-swap (`bswap`) is the last three rounds only, and field-swaps within packed structs use the identical two-mask pattern.
