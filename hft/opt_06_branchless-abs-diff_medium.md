## challenge: Branchless absolute difference |a - b|
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

`|a - b|` is a distance you compute constantly (price ticks apart, book depth). The naive `a > b ? a - b : b - a` branches on the ordering, and `a - b` can overflow a signed int (e.g. `INT_MIN - INT_MAX`). Implement `unsigned absdiff(int a, int b)` returning `|a - b|` as an `unsigned`, branchlessly and with no signed-overflow UB.

Constraints: `a`, `b` any 32-bit `int` including `INT_MIN`/`INT_MAX`. The true distance can be as large as `4294967295` (`|INT_MIN - INT_MAX|`), which fits exactly in `unsigned` but not in `int` — so compute in `unsigned`.

Example: `absdiff(5, 3)` → `2`. Example: `absdiff(3, 5)` → `2`. Example: `absdiff(INT_MIN, INT_MAX)` → `4294967295`. Example: `absdiff(0, INT_MIN)` → `2147483648`.

hint: Compute the raw wrapped difference `d = (unsigned)a - (unsigned)b` — it is either the answer or its two's-complement negation depending on which input is larger.
hint: Build a mask `m` from `a < b` and conditionally negate `d`: `(d ^ m) - m` flips the sign only when needed, all in `unsigned` so nothing overflows.
hint: `unsigned m = -(unsigned)(a < b);` is `0` when `a >= b` (keep `d`) and `0xFFFFFFFF` when `a < b` (negate `d`).

```cpp
// starter
unsigned absdiff(int a, int b);
```

```cpp
unsigned absdiff(int a, int b) {
    unsigned d = (unsigned)a - (unsigned)b;   // wrapped difference, no UB
    unsigned m = -(unsigned)(a < b);          // 0 or 0xFFFFFFFF
    return (d ^ m) - m;                        // negate d iff a < b
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b; unsigned want; } cases[] = {
        {5, 3, 2}, {3, 5, 2}, {7, 7, 0}, {-4, 4, 8}, {4, -4, 8},
        {INT_MIN, INT_MAX, 4294967295u}, {INT_MAX, INT_MIN, 4294967295u},
        {0, INT_MIN, 2147483648u}, {INT_MIN, 0, 2147483648u}, {-100, -100, 0},
    };
    for (auto& c : cases) {
        unsigned got = absdiff(c.a, c.b);
        if (got != c.want) { std::printf("absdiff(%d,%d)=%u want %u\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Doing the subtraction in `unsigned` makes the wrap well-defined: `d = (unsigned)a - (unsigned)b` is the correct magnitude when `a >= b` and its two's-complement negation when `a < b`. The mask `m = -(a < b)` is `0` or all-ones, and `(d ^ m) - m` is the conditional-negate idiom — it leaves `d` alone when `m == 0` and returns `~d + 1 == -d` when `m` is all-ones. Nothing is ever done in signed arithmetic, so `INT_MIN`/`INT_MAX` inputs never invoke overflow UB, and the full `4294967295` result is representable because it lives in `unsigned`. No branch means no ~15-20-cycle misprediction on irregular ordering; the compiler emits a `setcc`/`cmov`-style select. O(1).
