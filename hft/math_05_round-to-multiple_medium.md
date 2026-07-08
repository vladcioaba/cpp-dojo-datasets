## challenge: Round to the nearest multiple of a power of two
tags: bit-tricks, fast-math, alignment
track: hft
difficulty: medium

Quantizing a value to a grid (price levels, memory alignment) is a divide-then-multiply in the naive form. When the grid step `n` is a power of two, rounding to the nearest multiple is just a bias-and-mask. Implement `uint64_t round_to_multiple(uint64_t x, uint64_t n)` returning `x` rounded to the nearest multiple of `n` (ties round up), where `n` is a power of two. Use no `/`, `*`, or `%`.

Constraints: `n` is a power of two, `1 <= n <= 2^62`. `x + n/2 <= 2^64 - 1`.

Example: `round_to_multiple(11, 8) == 8`, `round_to_multiple(12, 8) == 16` (tie up), `round_to_multiple(13, 8) == 16`, `round_to_multiple(3, 8) == 0`, `round_to_multiple(x, 1) == x`.

hint: Rounding to nearest = shift the value up by half a step, then truncate down to a step boundary.
hint: Truncating down to a multiple of `n = 2^k` means clearing the low `k` bits: `& ~(n - 1)`.
hint: Add `n/2` (which is `n >> 1`) first so that a value exactly halfway lands on the upper multiple.

```cpp
// starter
#include <cstdint>
uint64_t round_to_multiple(uint64_t x, uint64_t n);
```

```cpp
uint64_t round_to_multiple(uint64_t x, uint64_t n) {
    return (x + (n >> 1)) & ~(n - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x, n, want; } cases[] = {
        {11,8,8},{12,8,16},{13,8,16},{3,8,0},{4,8,8},{0,8,0},{8,8,8},
        {100,1,100},{1000,1024,1024},{1600,1024,2048},{511,1024,0},{512,1024,1024},
        {(uint64_t)1<<62,(uint64_t)1<<62,(uint64_t)1<<62},
    };
    for (auto& c : cases) {
        uint64_t got = round_to_multiple(c.x, c.n);
        if (got != c.want) { std::printf("round_to_multiple(%llu,%llu)=%llu want %llu\n",
            (unsigned long long)c.x,(unsigned long long)c.n,
            (unsigned long long)got,(unsigned long long)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The naive `round(x / n) * n` costs a divide and a multiply. For power-of-two `n`, truncating down to a multiple is just masking off the low `k` bits (`& ~(n - 1)`), and rounding-to-nearest is achieved by pre-adding half the step (`n/2 = n >> 1`) so the truncation crosses to the next multiple exactly at the halfway point. Ties round up by construction. The whole thing is one add and one and — a couple of cycles.
