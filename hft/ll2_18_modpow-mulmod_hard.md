## challenge: Modular exponentiation without overflow
tags: integer-arithmetic, hot-path
track: hft
difficulty: hard

Symbol hashing, Lehmer RNG streams, and integrity checks all need `base^exp mod m` for 64-bit operands — but `a * b` overflows `uint64_t` long before the modulus does, silently corrupting the result. Implement two functions with no 128-bit types and no overflow anywhere: `uint64_t mulmod(uint64_t a, uint64_t b, uint64_t m)` computing `(a * b) mod m` by binary (shift-and-add) multiplication, and `uint64_t modpow(uint64_t base, uint64_t exp, uint64_t m)` computing `base^exp mod m` by square-and-multiply on top of it. Define `x^0 == 1` (so `modpow(0, 0, m) == 1`), and any result mod 1 is 0.

Constraints: `1 <= m < 2^63` (this bound is what makes overflow-free doubling possible — know why); any `a`, `b`, `base`, `exp`; `mulmod` is O(64), `modpow` is O(64) multiplies; use conditional subtraction, not `%`, inside the loops.

Example: `modpow(2, 10, 1000) == 24`, `modpow(7, 0, 13) == 1`, `mulmod(2^40, 2^40, 2^61 - 1) == 2^19` (since `2^61 ≡ 1` mod `2^61 - 1`).

hint: Russian-peasant multiplication: reduce `a`, `b` below `m` first; then for each low bit of `b`, conditionally add `a` into the result and double `a`, halving `b` each step — every add is of two values `< m`.
hint: With `m < 2^63`, both `r + a` and `a + a` are `< 2^64`, so they can't wrap; restore the invariant with `if (x >= m) x -= m;` — one compare-subtract, never a division.
hint: `modpow` is the same skeleton one level up: start `r = 1 % m` (handles `m == 1`), square `base` each step via `mulmod`, and multiply into `r` when the exponent bit is set.

```cpp
// starter
#include <cstdint>
uint64_t mulmod(uint64_t a, uint64_t b, uint64_t m);   // (a * b) % m, overflow-free
uint64_t modpow(uint64_t base, uint64_t exp, uint64_t m);  // base^exp % m
```

```cpp
uint64_t mulmod(uint64_t a, uint64_t b, uint64_t m) {
    a %= m;
    b %= m;
    uint64_t r = 0;
    while (b) {
        if (b & 1) {
            r += a;                    // r, a < m < 2^63: no wrap
            if (r >= m) r -= m;
        }
        a += a;                        // double, staying reduced
        if (a >= m) a -= m;
        b >>= 1;
    }
    return r;
}

uint64_t modpow(uint64_t base, uint64_t exp, uint64_t m) {
    uint64_t r = 1 % m;                // m == 1 -> everything is 0
    base %= m;
    while (exp) {
        if (exp & 1) r = mulmod(r, base, m);
        base = mulmod(base, base, m);
        exp >>= 1;
    }
    return r;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static uint64_t naivePow(uint64_t base, uint64_t exp, uint64_t m) {
    // Only called with small operands: products stay far below 2^64.
    uint64_t r = 1 % m;
    base %= m;
    for (uint64_t i = 0; i < exp; ++i) r = (r * base) % m;
    return r;
}
int main() {
    if (modpow(2, 10, 1000) != 24) { std::puts("2^10 mod 1000 must be 24"); return 1; }
    if (modpow(7, 0, 13) != 1) { std::puts("x^0 must be 1"); return 1; }
    if (modpow(5, 117, 1) != 0) { std::puts("mod 1 must be 0"); return 1; }
    if (mulmod(0, 12345, 97) != 0) { std::puts("0 * x must be 0"); return 1; }
    if (mulmod((1ull << 40), (1ull << 40), (1ull << 61) - 1) != (1ull << 19)) {
        std::puts("2^80 mod (2^61-1) must be 2^19"); return 1;
    }
    for (uint64_t b = 0; b < 8; ++b) {
        for (uint64_t e = 0; e < 8; ++e) {
            uint64_t got = modpow(b, e, 1000003ull);
            uint64_t want = naivePow(b, e, 1000003ull);
            if (got != want) {
                std::printf("modpow(%llu,%llu)=%llu want %llu\n",
                            (unsigned long long)b, (unsigned long long)e,
                            (unsigned long long)got, (unsigned long long)want);
                return 1;
            }
        }
    }
    // Fermat: for prime p, 2^(p-1) == 1 (mod p)
    if (modpow(2, 1000000006ull, 1000000007ull) != 1) { std::puts("Fermat check failed"); return 1; }
    // Big modulus near 2^62: x^(a+b) == x^a * x^b (mod m) must hold exactly
    uint64_t m0 = (1ull << 62) + 12345ull;
    uint64_t x = 0x123456789ABCDEFull;
    uint64_t lhs = modpow(x, 1000003ull + 777777ull, m0);
    uint64_t rhs = mulmod(modpow(x, 1000003ull, m0), modpow(x, 777777ull, m0), m0);
    if (lhs != rhs) { std::puts("exponent addition law violated at 2^62 scale"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The overflow analysis is the whole challenge. Once `a` and `b` are reduced below `m`, the loop maintains `r < m` and `a < m` as invariants; with `m < 2^63`, both `r + a` and `a + a` are strictly below `2^64`, so unsigned addition can't wrap — and since each sum is below `2m`, a single conditional subtract restores the invariant without ever touching the divider (a 64-bit `div` is 20–40 cycles; the compare-subtract pair is 1–2 and usually compiles to `cmov`). That bound is exactly why the contract stops at `2^63`: one bit of headroom is the price of the doubling trick. `mulmod` is Russian-peasant multiplication — decompose `b` into powers of two, accumulate `a * 2^i mod m` for each set bit — and `modpow` is the identical decomposition one level up: square-and-multiply over the exponent's bits, 64 squarings worst case, each of which is a 64-step mulmod, giving ~4096 simple ops for a full 64-bit exponentiation. `r = 1 % m` quietly handles the degenerate `m == 1` (everything is 0) and the `0^0 == 1` convention falls out of the loop never executing. In production you'd use `unsigned __int128` (gcc/clang) or Montgomery multiplication to make each product O(1); the shift-add ladder is the fully portable fallback — and the version you can write on a whiteboard while explaining, invariant by invariant, why no intermediate can ever overflow.
