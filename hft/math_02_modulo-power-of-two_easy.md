## challenge: Modulo by a power of two
tags: bit-tricks, fast-math
track: hft
difficulty: easy

Ring buffers and hash tables sized to a power of two let you replace an expensive `%` with a bit mask. Implement `uint64_t mod_pow2(uint64_t x, uint64_t n)` returning `x % n`, given that `n` is a power of two. Do not use `/` or `%`.

Constraints: `n` is a power of two, `1 <= n <= 2^63`. `0 <= x <= 2^64 - 1`.

Example: `mod_pow2(13, 8) == 5`, `mod_pow2(16, 8) == 0`, `mod_pow2(255, 16) == 15`, `mod_pow2(x, 1) == 0`.

hint: For `n = 2^k`, the remainder is exactly the low `k` bits of `x`.
hint: Subtracting one from a power of two yields a mask of all-ones below that bit: `n - 1 = 0b0111...1`.
hint: `x & (n - 1)` keeps precisely those low bits — no division unit involved.

```cpp
// starter
#include <cstdint>
uint64_t mod_pow2(uint64_t x, uint64_t n);
```

```cpp
uint64_t mod_pow2(uint64_t x, uint64_t n) {
    return x & (n - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x, n, want; } cases[] = {
        {13,8,5},{16,8,0},{255,16,15},{0,8,0},{7,1,0},
        {1024,1024,0},{1025,1024,1},{123456789,256,123456789ull & 255},
        {UINT64_MAX,(uint64_t)1<<63,(UINT64_MAX) & (((uint64_t)1<<63)-1)},
    };
    for (auto& c : cases) {
        uint64_t got = mod_pow2(c.x, c.n);
        if (got != c.want) { std::printf("mod_pow2(%llu,%llu)=%llu want %llu\n",
            (unsigned long long)c.x,(unsigned long long)c.n,
            (unsigned long long)got,(unsigned long long)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Division and modulo on x86 are 20-40+ cycle latency operations that also pipeline poorly; masking is a single cycle. It works only because a power of two `n = 2^k` gives a remainder equal to `x`'s low `k` bits, and `n - 1` is exactly the all-ones mask for those bits.
