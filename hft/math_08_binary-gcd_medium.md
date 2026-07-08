## challenge: Binary GCD (Stein's algorithm)
tags: fast-math, integer-math, bit-tricks
track: hft
difficulty: medium

Euclid's GCD leans on `%`, the slowest integer op. Stein's binary GCD replaces every modulo with subtractions and shifts. Implement `uint64_t gcd(uint64_t a, uint64_t b)` returning the greatest common divisor, with `gcd(0, b) == b` and `gcd(a, 0) == a`, using no `/` or `%`.

Constraints: `0 <= a, b <= 2^64 - 1`. No division or modulo.

Example: `gcd(12, 18) == 6`, `gcd(48, 36) == 12`, `gcd(17, 5) == 1`, `gcd(0, 9) == 9`, `gcd(2^60, 2^40) == 2^40`.

hint: Pull out common factors of two first: `gcd(2a, 2b) = 2*gcd(a, b)`; count the shared trailing zeros once with `__builtin_ctzll(a | b)`.
hint: If exactly one operand is even, that factor of two is not common — strip it, since it cannot divide the odd one.
hint: With both odd, `gcd(a, b) = gcd(|a - b| / 2, min(a, b))`; subtract smaller from larger, strip trailing zeros, and loop until one hits zero.

```cpp
// starter
#include <cstdint>
uint64_t gcd(uint64_t a, uint64_t b);
```

```cpp
uint64_t gcd(uint64_t a, uint64_t b) {
    if (a == 0) return b;
    if (b == 0) return a;
    int shift = __builtin_ctzll(a | b);
    a >>= __builtin_ctzll(a);
    do {
        b >>= __builtin_ctzll(b);
        if (a > b) { uint64_t t = a; a = b; b = t; }
        b -= a;
    } while (b != 0);
    return a << shift;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <numeric>
//__USER__
int main() {
    struct { uint64_t a, b, want; } cases[] = {
        {12,18,6},{48,36,12},{17,5,1},{0,9,9},{9,0,9},{0,0,0},{1,1,1},
        {(uint64_t)1<<60,(uint64_t)1<<40,(uint64_t)1<<40},
        {1000000007ull,998244353ull,1},
        {UINT64_MAX,1,1},{UINT64_MAX,UINT64_MAX,UINT64_MAX},
    };
    for (auto& c : cases) {
        uint64_t got = gcd(c.a, c.b);
        if (got != c.want) { std::printf("gcd(%llu,%llu)=%llu want %llu\n",
            (unsigned long long)c.a,(unsigned long long)c.b,
            (unsigned long long)got,(unsigned long long)c.want); return 1; }
    }
    for (uint64_t a = 0; a <= 255; ++a)
        for (uint64_t b = 0; b <= 255; ++b) {
            uint64_t got = gcd(a, b), want = std::gcd(a, b);
            if (got != want) { std::printf("gcd(%llu,%llu)=%llu want %llu\n",
                (unsigned long long)a,(unsigned long long)b,
                (unsigned long long)got,(unsigned long long)want); return 1; }
        }
    std::puts("PASS");
}
```

**Editorial:** Modulo is 20-40 cycles; Stein's algorithm uses only subtraction, comparison, and trailing-zero counts (one TZCNT/BSF each), which pipeline far better. Correctness rests on three identities: `gcd(2a, 2b) = 2*gcd(a, b)`; `gcd(2a, b) = gcd(a, b)` for odd `b`; and `gcd(a, b) = gcd(a - b, b)`. Removing the common factors of two up front and keeping both operands odd inside the loop guarantees each subtraction sheds at least one bit, so it terminates in O(bits) steps.
