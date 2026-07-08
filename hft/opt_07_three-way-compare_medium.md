## challenge: Branchless three-way compare
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

A three-way comparator returns `-1` if `a < b`, `0` if `a == b`, `+1` if `a > b` — the primitive under `<=>`, `qsort` comparators, and order-matching. The textbook version is a nest of branches; on unpredictable inputs each mispredict costs ~15-20 cycles. Implement `int cmp3(int a, int b)` returning `-1/0/+1` with no branches and no overflow.

Constraints: `a`, `b` any 32-bit `int` including `INT_MIN`/`INT_MAX`. Do not compute `a - b` (it overflows for far-apart inputs) — compare directly.

Example: `cmp3(3, 5)` → `-1`. Example: `cmp3(5, 3)` → `1`. Example: `cmp3(7, 7)` → `0`. Example: `cmp3(INT_MIN, INT_MAX)` → `-1`.

hint: Two independent comparisons, each `0`/`1`, can be subtracted so the result is exactly one of `-1`, `0`, `1` — and neither compares by subtracting the operands.
hint: `(a > b)` is `1` only when `a` is greater; `(a < b)` is `1` only when `a` is smaller; they are never both `1`.
hint: `(a > b) - (a < b)` gives `+1`, `-1`, or `0` directly.

```cpp
// starter
int cmp3(int a, int b);
```

```cpp
int cmp3(int a, int b) {
    return (a > b) - (a < b);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b, want; } cases[] = {
        {3, 5, -1}, {5, 3, 1}, {7, 7, 0}, {-1, 1, -1}, {1, -1, 1},
        {INT_MIN, INT_MAX, -1}, {INT_MAX, INT_MIN, 1}, {INT_MIN, INT_MIN, 0},
        {INT_MAX, INT_MAX, 0}, {0, -1, 1},
    };
    for (auto& c : cases) {
        int got = cmp3(c.a, c.b);
        if (got != c.want) { std::printf("cmp3(%d,%d)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The trap is `sign(a - b)`: the subtraction overflows for far-apart signed inputs (`INT_MIN - INT_MAX` is UB and wraps to a positive value, giving the wrong sign). Comparing directly avoids it — `(a > b)` and `(a < b)` each evaluate to `0` or `1`, are mutually exclusive, and their difference is exactly `+1`, `-1`, or `0`. No subtraction of operands means no overflow even at the extremes; no branch means the irregular comparison stream never triggers a ~15-20-cycle pipeline flush. A compiler lowers each comparison to a `setcc` and emits one subtract, so `cmp3` is a few jump-free dependent instructions. O(1).
