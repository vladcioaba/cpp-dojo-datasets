## challenge: Round to powers of two, both ways
tags: bit-tricks, hot-path
track: hft
difficulty: medium

Ring buffer capacities, arena sizes, and hash table sizes must be powers of two so that indexing is a mask instead of a division. Implement both directions for 64-bit values, without loops over bits and without `std::bit_ceil`/`std::bit_floor`/builtins: `uint64_t floorPow2(uint64_t x)` returns the largest power of two `<= x` (define `floorPow2(0) == 0`), and `uint64_t ceilPow2(uint64_t x)` returns the smallest power of two `>= x` for `1 <= x <= 2^63`.

Constraints: O(1) — a fixed ladder of shifts and ORs (6 steps for 64 bits); no loops, no lookup tables, no builtins.

Example: `floorPow2(1000) == 512`, `floorPow2(1024) == 1024`, `floorPow2(0) == 0`. `ceilPow2(1000) == 1024`, `ceilPow2(1024) == 1024`, `ceilPow2(1) == 1`.

hint: The bit-smear ladder `x |= x >> 1; x |= x >> 2; ... x |= x >> 32;` propagates the most significant set bit into every position below it, turning `x` into `2^(k+1) - 1` where `k` is the MSB index.
hint: After smearing, `x - (x >> 1)` leaves just the top bit — that's the floor. For zero input the smear leaves zero, so the definition `floorPow2(0) == 0` falls out for free.
hint: For the ceiling, subtract 1 *before* smearing and add 1 after — the decrement is what makes exact powers of two map to themselves instead of doubling.

```cpp
// starter
#include <cstdint>
uint64_t floorPow2(uint64_t x);
uint64_t ceilPow2(uint64_t x);
```

```cpp
uint64_t floorPow2(uint64_t x) {
    x |= x >> 1;
    x |= x >> 2;
    x |= x >> 4;
    x |= x >> 8;
    x |= x >> 16;
    x |= x >> 32;
    return x - (x >> 1);   // keep only the top set bit
}

uint64_t ceilPow2(uint64_t x) {
    x -= 1;                // so exact powers stay put
    x |= x >> 1;
    x |= x >> 2;
    x |= x >> 4;
    x |= x >> 8;
    x |= x >> 16;
    x |= x >> 32;
    return x + 1;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x, want; } fl[] = {
        {0ull, 0ull}, {1ull, 1ull}, {2ull, 2ull}, {3ull, 2ull}, {5ull, 4ull},
        {6ull, 4ull}, {1000ull, 512ull}, {1024ull, 1024ull}, {1025ull, 1024ull},
        {(1ull << 63), (1ull << 63)}, {(1ull << 63) + 5ull, (1ull << 63)},
        {0xFFFFFFFFFFFFFFFFull, (1ull << 63)},
    };
    for (auto& c : fl) {
        uint64_t got = floorPow2(c.x);
        if (got != c.want) {
            std::printf("floorPow2(%llu)=%llu want %llu\n",
                        (unsigned long long)c.x, (unsigned long long)got, (unsigned long long)c.want);
            return 1;
        }
    }
    struct { uint64_t x, want; } ce[] = {
        {1ull, 1ull}, {2ull, 2ull}, {3ull, 4ull}, {5ull, 8ull}, {1000ull, 1024ull},
        {1023ull, 1024ull}, {1024ull, 1024ull}, {1025ull, 2048ull},
        {(1ull << 62) + 1ull, (1ull << 63)}, {(1ull << 63), (1ull << 63)},
    };
    for (auto& c : ce) {
        uint64_t got = ceilPow2(c.x);
        if (got != c.want) {
            std::printf("ceilPow2(%llu)=%llu want %llu\n",
                        (unsigned long long)c.x, (unsigned long long)got, (unsigned long long)c.want);
            return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** The smear ladder is a doubling propagation: after `x |= x >> 1` the MSB covers 2 positions, after `>> 2` it covers 4, and after the `>> 32` step every bit at or below the MSB is set — six steps handle 64 bits because coverage doubles each time. From the smeared value `2^(k+1)-1`, the floor is `x - (x >> 1)` (all-ones minus all-ones-shifted leaves the top bit), and the ceiling comes from the classic decrement/smear/increment: subtracting 1 first means an exact power like 1024 smears from 1023 and increments right back to 1024, while 1025 smears from 1024 up to 2047 and lands on 2048. Watch the domain edges — `ceilPow2` overflows to 0 for `x > 2^63` (no 64-bit power of two exists above it), which is why the constraint stops at `2^63`; and `floorPow2(0) == 0` falls out naturally since zero smears to zero. Hardware does this with one `lzcnt`, and C++20 exposes it as `std::bit_ceil`/`std::bit_floor` — but the ladder is what you write where those don't reach (constexpr-in-C++17 code, GPUs, verification models), and deriving it shows you understand *why* power-of-two capacities matter: `index & (cap - 1)` replaces a 20–40 cycle divide with a 1-cycle AND on every queue operation.
