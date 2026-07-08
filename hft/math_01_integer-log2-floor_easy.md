## challenge: Integer log2 (floor)
tags: bit-tricks, fast-math
track: hft
difficulty: easy

On the hot path you often need `floor(log2(x))` — to pick a size class, a bucket, or a shift amount — and calling `std::log2` drags in the FPU and its rounding surprises. Implement `int ilog2(uint64_t x)` returning the position of the highest set bit, i.e. `floor(log2(x))`. `x` is guaranteed non-zero.

Constraints: `1 <= x <= 2^64 - 1`. No floating point.

Example: `ilog2(1) == 0`, `ilog2(2) == 1`, `ilog2(3) == 1`, `ilog2(255) == 7`, `ilog2(256) == 8`.

hint: The floor of log2 is just the index of the most significant set bit — no arithmetic needed, only a bit scan.
hint: Hardware exposes a "count leading zeros" instruction; GCC/Clang surface it as `__builtin_clzll`.
hint: In a 64-bit word the MSB index is `63 - clz(x)`; that is why `x` must be non-zero, since `clz(0)` is undefined.

```cpp
// starter
#include <cstdint>
int ilog2(uint64_t x);
```

```cpp
int ilog2(uint64_t x) {
    return 63 - __builtin_clzll(x);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t x; int want; } cases[] = {
        {1,0},{2,1},{3,1},{4,2},{7,2},{8,3},{255,7},{256,8},
        {1023,9},{1024,10},{(uint64_t)1<<62,62},{(uint64_t)1<<63,63},
        {UINT64_MAX,63},
    };
    for (auto& c : cases) {
        int got = ilog2(c.x);
        if (got != c.want) { std::printf("ilog2(%llu)=%d want %d\n",
            (unsigned long long)c.x, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `floor(log2(x))` equals the index of the highest set bit. `__builtin_clzll` returns the count of leading zero bits, so `63 - clz` is that index — a single hardware instruction (BSR/LZCNT, ~1-3 cycles) versus a `std::log2` call that goes through the FPU (tens of cycles) and can misround right at exact powers of two. It is undefined for `x == 0`, hence the non-zero constraint.
