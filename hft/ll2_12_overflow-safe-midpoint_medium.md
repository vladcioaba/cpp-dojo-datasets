## challenge: Midpoint without overflow
tags: integer-arithmetic, bit-tricks, hot-path
track: hft
difficulty: medium

`(a + b) / 2` is a latent bug: the sum overflows for large operands — signed overflow is undefined behavior — and even when it doesn't, integer division truncates toward zero, not toward negative infinity. Implement `int32_t floorMid(int32_t a, int32_t b)` returning exactly `floor((a + b) / 2)` for *all* `int32_t` pairs, including `INT_MAX` with `INT_MAX` and mixed signs, with no overflow, no UB, and no widening to 64-bit.

Constraints: any `int32_t` values for `a` and `b`; 32-bit arithmetic only (no `int64_t` anywhere); branchless — shifts, ANDs, adds.

Example: `floorMid(3, 4) == 3`, `floorMid(-3, -4) == -4` (floor of -3.5), `floorMid(INT_MAX, INT_MAX) == INT_MAX`, `floorMid(INT_MAX, INT_MIN) == -1`.

hint: Halve each operand separately: since C++20, `a >> 1` on a signed value is guaranteed arithmetic shift, which is exactly `floor(a / 2)` — for negatives too (unlike `a / 2`, which truncates toward zero).
hint: `(a >> 1) + (b >> 1)` can never overflow (each half is at most half the range), but it drops half a unit from each odd operand — half plus half is the whole unit you're missing.
hint: The two dropped halves add back to exactly 1 precisely when *both* operands are odd: add `a & b & 1`.

```cpp
// starter
#include <cstdint>
int32_t floorMid(int32_t a, int32_t b);
```

```cpp
int32_t floorMid(int32_t a, int32_t b) {
    // a>>1 is floor(a/2) in C++20 (arithmetic shift); the halves each drop 0.5
    // when odd, and those two halves sum to a whole 1 exactly when both are odd.
    return (a >> 1) + (b >> 1) + (a & b & 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <climits>
//__USER__
static int32_t ref(int32_t a, int32_t b) {
    long long s = (long long)a + (long long)b;
    return (int32_t)(s >> 1);   // arithmetic shift of the wide sum == floor((a+b)/2)
}
int main() {
    struct { int32_t a, b; } cases[] = {
        {INT_MAX, INT_MAX}, {INT_MIN, INT_MIN}, {INT_MAX, INT_MIN}, {INT_MIN, INT_MAX},
        {3, 4}, {4, 3}, {3, 5}, {-3, -4}, {-3, 4}, {3, -4}, {-3, -5},
        {0, 0}, {-1, 0}, {0, -1}, {1, 1}, {-1, -1},
        {INT_MAX - 1, INT_MAX}, {INT_MIN, INT_MIN + 1}, {INT_MIN + 1, INT_MAX},
    };
    for (auto& c : cases) {
        int32_t got = floorMid(c.a, c.b);
        int32_t want = ref(c.a, c.b);
        if (got != want) {
            std::printf("floorMid(%d,%d)=%d want %d\n", c.a, c.b, got, want);
            return 1;
        }
    }
    for (int a = -9; a <= 9; ++a) {
        for (int b = -9; b <= 9; ++b) {
            if (floorMid(a, b) != ref(a, b)) {
                std::printf("floorMid(%d,%d)=%d want %d\n", a, b, floorMid(a, b), ref(a, b));
                return 1;
            }
        }
    }
    std::puts("PASS");
}
```

**Editorial:** Decompose each operand as `2*(x >> 1) + (x & 1)`: the shift is floor-halving (C++20 finally *guarantees* arithmetic right shift on signed values — before that it was implementation-defined, though universal in practice), and the low bit is the remainder. Then `floor((a+b)/2) = (a>>1) + (b>>1) + floor(((a&1)+(b&1))/2)`, and that last term is 1 only when both low bits are 1 — i.e. `a & b & 1`. Every intermediate stays comfortably in range: each half is within `[-2^30, 2^30)`, so the sum can't overflow, and no UB is possible. Contrast the classics that fail: `(a+b)/2` overflows (the bug that famously lurked in binary searches for decades as `mid = (lo+hi)/2`); `a + (b-a)/2` fixes overflow only when the difference fits, and rounds toward `a`, not toward negative infinity; `std::midpoint` is overflow-safe but also rounds toward `a` — a different contract. Floor-rounding matters when the midpoint feeds price arithmetic that must be monotonic: floor is order-preserving under negation offsets, truncation is not. Three ALU ops, branchless, and correct on the entire domain — exactly the kind of primitive you prove once and reuse everywhere.
