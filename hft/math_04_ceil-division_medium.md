## challenge: Ceil division without floats
tags: fast-math, integer-math
track: hft
difficulty: medium

You need `ceil(a / b)` — say, the number of fixed-size blocks to cover a byte count — but `(uint64_t)ceil((double)a / b)` pays for a float divide and silently misrounds once `a` exceeds `2^53`. Implement `uint64_t ceil_div(uint64_t a, uint64_t b)` using only integer arithmetic.

Constraints: `0 <= a`, `1 <= b`, and `a + b - 1` fits in `uint64` (i.e. `a <= 2^64 - b`). One integer divide is allowed.

Example: `ceil_div(7, 3) == 3`, `ceil_div(6, 3) == 2`, `ceil_div(0, 5) == 0`, `ceil_div(1, 1000) == 1`, `ceil_div(10^18, 7) == 142857142857142858`.

hint: Integer `/` already floors; you want to floor a value that has been nudged up by just under one whole `b`.
hint: Adding `b - 1` before dividing turns any nonzero remainder into one extra count, while an exact multiple is left unchanged.
hint: Guard the add against overflow — `(a + b - 1) / b` is valid only while `a + b - 1` does not wrap; if `a` can be near the max, use `a / b + (a % b != 0)` instead.

```cpp
// starter
#include <cstdint>
uint64_t ceil_div(uint64_t a, uint64_t b);
```

```cpp
uint64_t ceil_div(uint64_t a, uint64_t b) {
    return (a + b - 1) / b;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint64_t a, b, want; } cases[] = {
        {7,3,3},{6,3,2},{9,3,3},{0,5,0},{1,1000,1},{1000,1,1000},
        {8,8,1},{9,8,2},{1,1,1},
        {1000000000000000000ull,7,142857142857142858ull},
        {1000000000000000000ull,1000000000ull,1000000000ull},
    };
    for (auto& c : cases) {
        uint64_t got = ceil_div(c.a, c.b);
        if (got != c.want) { std::printf("ceil_div(%llu,%llu)=%llu want %llu\n",
            (unsigned long long)c.a,(unsigned long long)c.b,
            (unsigned long long)got,(unsigned long long)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** One integer divide is unavoidable for a runtime divisor, but this eliminates the float round-trip and its precision loss (a `double` has only 53 mantissa bits, so `ceil((double)a / b)` misrounds above `2^53`). Adding `b - 1` lifts any partial block into the next integer; an exact multiple `a = k*b` gives `(k*b + b - 1) / b = k`. The only caveat is overflow of `a + b - 1`; when `a` can approach the max, `a / b + (a % b != 0)` is the overflow-proof form.
