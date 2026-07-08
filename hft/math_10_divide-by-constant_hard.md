## challenge: Divide by a constant via reciprocal multiply
tags: fast-math, integer-math, reciprocal
track: hft
difficulty: hard

A `div` instruction is ~20-40 cycles; when the divisor is a compile-time constant, compilers replace it with a multiply by a fixed-point reciprocal plus a shift. Reproduce that by hand for the classic case: implement `uint32_t div3(uint32_t x)` returning `x / 3` for every 32-bit `x`, using only a multiply and a shift (no `/` or `%`).

Constraints: `0 <= x <= 2^32 - 1`. Exactly one 64-bit multiply and one shift; no division or modulo.

Example: `div3(0) == 0`, `div3(2) == 0`, `div3(3) == 1`, `div3(6) == 2`, `div3(3000000000) == 1000000000`, `div3(2^32 - 1) == 1431655765`.

hint: `1/3` in binary is `0.01010101...`; approximate it as a fixed-point reciprocal `m / 2^s`, and the divide becomes a multiply then a shift.
hint: Use the magic constant `m = ceil(2^33 / 3) = 0xAAAAAAAB` with shift `s = 33`; the round-up in the reciprocal keeps it exact for every `uint32`.
hint: Do the multiply in `uint64` (`(uint64_t)x * 0xAAAAAAABull`) so the product's high bits survive, then `>> 33`.

```cpp
// starter
#include <cstdint>
uint32_t div3(uint32_t x);
```

```cpp
uint32_t div3(uint32_t x) {
    return (uint32_t)(((uint64_t)x * 0xAAAAAAABull) >> 33);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static int check(uint64_t x) {
    uint32_t got = div3((uint32_t)x), want = (uint32_t)x / 3u;
    if (got != want) { std::printf("div3(%llu)=%u want %u\n",
        (unsigned long long)x, got, want); return 1; }
    return 0;
}
int main() {
    uint64_t fixed[] = {0,1,2,3,6,9,3000000000ull,4294967295ull,2863311530ull,
                        2863311531ull,1431655764ull,1431655765ull};
    for (uint64_t x : fixed) if (check(x)) return 1;
    for (uint64_t x = 0; x < 4000000ull; ++x) if (check(x)) return 1;
    for (uint64_t x = 4290000000ull; x <= 4294967295ull; ++x) if (check(x)) return 1;
    for (uint64_t x = 0; x <= 4294967295ull; x += 99991ull) if (check(x)) return 1;
    std::puts("PASS");
}
```

**Editorial:** General magic-number division. For a divisor `d`, precompute `m = ceil(2^(N+s) / d)` so that `floor(x*m / 2^(N+s)) == floor(x / d)` for all `N`-bit `x`; for `d = 3`, `N = 32`, the constant `m = 0xAAAAAAAB` (`= ceil(2^33 / 3)`) with shift `33` works across the entire range. The round-up in `m` compensates for the truncated reciprocal so it never underestimates. Cost is one multiply (~3-4 cycles) plus a shift versus a 20-40 cycle `div` — exactly what `-O2` emits for `x / 3`. To also get `x % 3`, compute `x - 3 * div3(x)`.
