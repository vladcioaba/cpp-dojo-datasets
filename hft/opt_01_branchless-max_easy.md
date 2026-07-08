## challenge: Branchless max
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: easy

A data-dependent branch that the CPU cannot predict costs ~15-20 cycles when it mispredicts and flushes the pipeline. Compute the maximum of two ints without a branch, using the two's-complement bit trick: `a ^ ((a ^ b) & -(a < b))`. Implement `int bmax(int a, int b)` returning the larger of the two. (The compiler will usually emit a `cmov` for `a < b ? a : b` — but interviewers want you to derive the mask by hand.)

Constraints: `a` and `b` are any 32-bit `int`, including `INT_MIN` and `INT_MAX`. No overflow is allowed — you are only comparing and selecting, never adding.

Example: `bmax(3, 5)` → `5`. Example: `bmax(-2, -9)` → `-2`. Example: `bmax(INT_MIN, INT_MAX)` → `INT_MAX`.

hint: Turn the boolean `a < b` into an all-zero or all-ones bitmask with unary minus, then use it to select between the two inputs — no jump means nothing to mispredict.
hint: `-(a < b)` is `0` (all bits clear) when `a >= b` and `-1` (all bits set) when `a < b`.
hint: `a ^ ((a ^ b) & mask)` collapses to `a` when the mask is `0` and to `b` when it is all-ones; pick which one you keep so the result is the larger value.

```cpp
// starter
int bmax(int a, int b);
```

```cpp
int bmax(int a, int b) {
    return a ^ ((a ^ b) & -(a < b));
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b, want; } cases[] = {
        {3, 5, 5}, {5, 3, 5}, {-2, -9, -2}, {7, 7, 7},
        {0, -1, 0}, {INT_MIN, INT_MAX, INT_MAX}, {INT_MAX, 0, INT_MAX},
        {INT_MIN, INT_MIN, INT_MIN}, {-100, -100, -100}, {INT_MIN, 0, 0},
    };
    for (auto& c : cases) {
        int got = bmax(c.a, c.b);
        if (got != c.want) { std::printf("bmax(%d,%d)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** `a < b` produces `0` or `1`; unary minus turns that into an all-zero or all-ones mask. `a ^ ((a ^ b) & mask)` equals `a` when the mask is `0` (so `a >= b`, and `a` is the max) and equals `a ^ (a ^ b) == b` when the mask is all-ones (so `a < b`, and `b` is the max). There is no conditional jump, so the branch predictor is never involved and there is no ~15-20-cycle flush on misprediction. In practice a modern compiler already lowers `a < b ? a : b` to a `cmov` at `-O2`; the mask form makes the data-flow explicit and is the answer expected in interviews. Constant O(1) work, no overflow because nothing is added.
