## challenge: Q16.16 fixed-point multiply
tags: fixed-point, fast-math, overflow
track: hft
difficulty: medium

Fixed-point keeps fractional math deterministic and float-free. In Q16.16 a real number `v` is stored as the `int32` raw `round(v * 65536)` (16 integer bits, 16 fractional bits). Multiplying two Q16.16 raws naively as `int32` overflows and lands in the wrong scale. Implement `int32_t qmul(int32_t a, int32_t b)` returning the Q16.16 product, using a 64-bit intermediate.

Constraints: inputs are Q16.16 raws in `INT32_MIN..INT32_MAX`; the true product fits in Q16.16. Truncate toward negative infinity (arithmetic shift).

Example: with `1.0 = 65536`, `2.0 = 131072`, `1.5 = 98304`: `qmul(65536, 131072) == 131072` (1.0*2.0), `qmul(98304, 98304) == 147456` (1.5*1.5 = 2.25), `qmul(-98304, 98304) == -147456`.

hint: Two Q16.16 values multiplied give a Q32.32 number — the binary point moves to bit 32, so you must shift back down by 16.
hint: `int32 * int32` can need 62 bits; form the product in `int64` before shifting.
hint: The scaled result is `((int64_t)a * b) >> 16`; in C++20 a signed right shift is arithmetic, giving floor (truncation toward -inf).

```cpp
// starter
#include <cstdint>
int32_t qmul(int32_t a, int32_t b);
```

```cpp
int32_t qmul(int32_t a, int32_t b) {
    return (int32_t)(((int64_t)a * b) >> 16);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { int32_t a, b, want; } cases[] = {
        {65536,131072,131072},     // 1.0 * 2.0 = 2.0
        {98304,98304,147456},      // 1.5 * 1.5 = 2.25
        {-98304,98304,-147456},    // -1.5 * 1.5 = -2.25
        {-98304,-98304,147456},    // -1.5 * -1.5 = 2.25
        {196608,16384,49152},      // 3.0 * 0.25 = 0.75
        {0,123456,0},
        {65536,65536,65536},       // 1.0 * 1.0 = 1.0
        {16711680,131072,33423360},// 255.0 * 2.0 = 510.0
    };
    for (auto& c : cases) {
        int32_t got = qmul(c.a, c.b);
        if (got != c.want) { std::printf("qmul(%d,%d)=%d want %d\n",
            (int)c.a,(int)c.b,(int)got,(int)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Multiplying `Qm.f` by `Qm.f` yields `Q(2m).(2f)`; the binary point is now `2f = 32` bits up, so `>> 16` realigns to Q16.16 and divides out the doubled scale in one step. The 64-bit intermediate is mandatory — two 32-bit operands need up to 62 result bits. For round-to-nearest instead of truncation, add `(1 << 15)` before the shift. Everything is integer: no FPU, and bit-for-bit reproducible across machines, which matters for deterministic pricing.
