## challenge: Integer square root without floating point
tags: fast-math, integer-math, bit-tricks
track: hft
difficulty: hard

`(uint64_t)sqrt((double)x)` can be off by one near perfect squares — a `double`'s 53-bit mantissa cannot represent every 64-bit integer — and the FPU round-trip is slow. Implement `uint64_t isqrt(uint64_t x)` returning `floor(sqrt(x))` exactly over the full unsigned 64-bit range, using only integer operations.

Constraints: `0 <= x <= 2^64 - 1`. The result `r` satisfies `r*r <= x < (r+1)*(r+1)` (reason about the upper bound; `(r+1)^2` may overflow if computed naively).

Example: `isqrt(0) == 0`, `isqrt(1) == 1`, `isqrt(15) == 3`, `isqrt(16) == 4`, `isqrt(24) == 4`, `isqrt(2^64 - 1) == 4294967295`.

hint: Build the root two bits of radicand at a time, most-significant first, like long division for square roots.
hint: Maintain a "bit" that walks down the powers of four (`1<<62`, `1<<60`, ...); start it at the largest power of four not exceeding `x`.
hint: At each step test whether the current root plus the candidate bit still squares within the remainder; the update `res = (res >> 1) + bit` advances the running root with no multiply.

```cpp
// starter
#include <cstdint>
uint64_t isqrt(uint64_t x);
```

```cpp
uint64_t isqrt(uint64_t x) {
    uint64_t res = 0;
    uint64_t bit = (uint64_t)1 << 62;
    while (bit > x) bit >>= 2;
    while (bit) {
        if (x >= res + bit) {
            x -= res + bit;
            res = (res >> 1) + bit;
        } else {
            res >>= 1;
        }
        bit >>= 2;
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x, want; } fixed[] = {
        {0,0},{1,1},{2,1},{3,1},{4,2},{8,2},{15,3},{16,4},{24,4},{25,5},
        {18446744065119617025ull,4294967295ull},        // 4294967295^2
        {18446744065119617025ull+1,4294967295ull},
        {UINT64_MAX,4294967295ull},
        {4000000000000000000ull,2000000000ull},          // 2e9^2 = 4e18
    };
    for (auto& c : fixed) {
        uint64_t got = isqrt(c.x);
        if (got != c.want) { std::printf("isqrt(%llu)=%llu want %llu\n",
            (unsigned long long)c.x,(unsigned long long)got,(unsigned long long)c.want); return 1; }
    }
    // dense sweep: r tracked incrementally, no overflow (r small)
    uint64_t r = 0;
    for (uint64_t x = 0; x <= 300000; ++x) {
        while ((r + 1) * (r + 1) <= x) ++r;
        if (isqrt(x) != r) { std::printf("sweep isqrt(%llu)=%llu want %llu\n",
            (unsigned long long)x,(unsigned long long)isqrt(x),(unsigned long long)r); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Bit-by-bit "digit at a time" square root. Processing two bits per step (`bit` moves by `>>2` over powers of four) mirrors decimal long-division square root in base 4. The invariant keeps `res` as the partial root, and the test `x >= res + bit` decides each bit without ever multiplying. It is exact across all `2^64` values, unlike the `double` path which loses precision above `2^53` and needs a correction step. Cost is 32 iterations of adds and shifts, fully integer.
