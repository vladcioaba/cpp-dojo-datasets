## challenge: Saturating multiply (int32)
tags: fast-math, overflow, integer-math
track: hft
difficulty: medium

In fixed-point accumulation you often want a multiply that clamps to the representable range instead of wrapping — signed overflow is UB and a correctness disaster for prices. Implement `int32_t sat_mul(int32_t a, int32_t b)` returning `a * b` clamped to `[INT32_MIN, INT32_MAX]`, with no signed overflow anywhere.

Constraints: full `int32` inputs. The result is clamped to the `int32` range.

Example: `sat_mul(100, 100) == 10000`, `sat_mul(100000, 100000) == 2147483647`, `sat_mul(-100000, 100000) == -2147483648`, `sat_mul(INT32_MIN, -1) == 2147483647` (true value `2^31` overflows), `sat_mul(0, INT32_MIN) == 0`.

hint: The product of two 32-bit ints always fits in 64 bits, so compute it wide first — no overflow there.
hint: Then a single pair of comparisons against `INT32_MAX` / `INT32_MIN` decides whether to clamp.
hint: Watch the asymmetric corner `INT32_MIN * -1 = 2^31`, one past `INT32_MAX` — the wide compare catches it, whereas a naive 32-bit multiply is UB.

```cpp
// starter
#include <cstdint>
int32_t sat_mul(int32_t a, int32_t b);
```

```cpp
int32_t sat_mul(int32_t a, int32_t b) {
    int64_t p = (int64_t)a * b;
    if (p > INT32_MAX) return INT32_MAX;
    if (p < INT32_MIN) return INT32_MIN;
    return (int32_t)p;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { int32_t a, b, want; } cases[] = {
        {100,100,10000},{0,INT32_MIN,0},{1,INT32_MAX,INT32_MAX},
        {100000,100000,INT32_MAX},{-100000,100000,INT32_MIN},
        {INT32_MIN,-1,INT32_MAX},{-1,INT32_MIN,INT32_MAX},
        {INT32_MAX,INT32_MAX,INT32_MAX},{INT32_MIN,INT32_MIN,INT32_MAX},
        {46340,46340,2147395600},{46341,46341,INT32_MAX},
        {-46341,46341,INT32_MIN},
    };
    for (auto& c : cases) {
        int32_t got = sat_mul(c.a, c.b);
        if (got != c.want) { std::printf("sat_mul(%d,%d)=%d want %d\n",
            (int)c.a,(int)c.b,(int)got,(int)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The key realization is that `int32 * int32` is at most `2^62` in magnitude, so one `int64` multiply cannot overflow and gives the exact product; clamping is then two branches (often compiled to branch-free `cmov`). The naive `int32 r = a * b` is undefined on overflow and, even if it wrapped, would silently corrupt. The nasty corner is `INT32_MIN * -1 = +2^31`, exactly one above `INT32_MAX`, which the wide comparison saturates correctly.
