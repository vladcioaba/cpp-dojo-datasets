## challenge: Round up to power of two
tags: bit-tricks, ring-buffer
track: hft

Ring-buffer and hash-table sizes are powers of two so the modulo becomes a mask. Implement `uint64_t next_pow2(uint64_t x)` returning the smallest power of two that is `>= x`, for `2 <= x <= 2^62`. Bit-smear then increment.

```cpp
uint64_t next_pow2(uint64_t x) {
    --x;
    x |= x >> 1;  x |= x >> 2;  x |= x >> 4;
    x |= x >> 8;  x |= x >> 16; x |= x >> 32;
    return x + 1;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x, want; } cases[] = {
        {2, 2}, {3, 4}, {5, 8}, {8, 8}, {9, 16},
        {1000, 1024}, {1u << 20, 1u << 20}, {(1u << 20) + 1, 1u << 21},
        {(uint64_t)1 << 61, (uint64_t)1 << 61}, {((uint64_t)1 << 61) + 1, (uint64_t)1 << 62},
    };
    for (auto& c : cases) {
        uint64_t got = next_pow2(c.x);
        if (got != c.want) { std::printf("next_pow2(%llu)=%llu want %llu\n",
            (unsigned long long)c.x, (unsigned long long)got, (unsigned long long)c.want); return 1; }
    }
    std::puts("PASS");
}
```
