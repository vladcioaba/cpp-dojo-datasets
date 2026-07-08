## challenge: Branchless is-in-range with the unsigned trick
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

Testing `lo <= x && x <= hi` is two comparisons and, with `&&`, a short-circuit branch. On the hot path (validating an index, gating a price band) the branch mispredicts for ~15-20 cycles on irregular inputs. Implement `bool in_range(int x, int lo, int hi)` using the classic single-comparison unsigned trick, with no branch and no signed-overflow UB.

Constraints: `lo <= hi`. All of `x`, `lo`, `hi` any 32-bit `int` including `INT_MIN`/`INT_MAX`. The offsets `x - lo` and `hi - lo` can exceed `INT_MAX`, so compute them in `unsigned`.

Example: `in_range(5, 0, 10)` → `true`. Example: `in_range(-1, 0, 10)` → `false`. Example: `in_range(11, 0, 10)` → `false`. Example: `in_range(0, -5, 5)` → `true`.

hint: Shift the window so it starts at zero: rebase `x` and `hi` by `lo`. Anything below `lo` wraps around to a huge unsigned value and falls outside in a single test.
hint: In `unsigned`, `x < lo` makes `(unsigned)x - (unsigned)lo` wrap to a value larger than `(unsigned)hi - (unsigned)lo`, so one comparison covers both ends.
hint: `return (unsigned)((unsigned)x - (unsigned)lo) <= (unsigned)((unsigned)hi - (unsigned)lo);`.

```cpp
// starter
bool in_range(int x, int lo, int hi);
```

```cpp
bool in_range(int x, int lo, int hi) {
    return ((unsigned)x - (unsigned)lo) <= ((unsigned)hi - (unsigned)lo);
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int x, lo, hi; bool want; } cases[] = {
        {5, 0, 10, true}, {-1, 0, 10, false}, {11, 0, 10, false},
        {0, 0, 10, true}, {10, 0, 10, true}, {0, -5, 5, true}, {-6, -5, 5, false},
        {INT_MIN, INT_MIN, INT_MAX, true}, {INT_MAX, 0, INT_MAX, true},
        {INT_MIN, 0, INT_MAX, false},
    };
    for (auto& c : cases) {
        bool got = in_range(c.x, c.lo, c.hi);
        if (got != c.want) { std::printf("in_range(%d,%d,%d)=%d want %d\n", c.x, c.lo, c.hi, (int)got, (int)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Rebasing the window to zero collapses a two-sided test into one. In `unsigned`, `(unsigned)x - (unsigned)lo` is the offset from `lo`; if `x < lo` the subtraction wraps to a value near `UINT_MAX`, which is guaranteed greater than the width `(unsigned)hi - (unsigned)lo`, so the single `<=` rejects both the below-`lo` and above-`hi` cases at once. Doing it in `unsigned` also sidesteps signed-overflow UB when `hi - lo` or `x - lo` exceeds `INT_MAX` (e.g. the full `[INT_MIN, INT_MAX]` window). One comparison, no `&&` short-circuit, so no ~15-20-cycle misprediction on irregular data — and half the comparison work of the naive form. O(1).
