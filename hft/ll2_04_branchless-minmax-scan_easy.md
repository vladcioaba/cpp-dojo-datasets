## challenge: Branchless min/max scan
tags: simd, branch-prediction, hot-path
track: hft
difficulty: easy

Find the minimum and maximum of an array in one pass with no unpredictable branches. Implement `MinMax scanMinMax(const int32_t* a, size_t n)` returning `struct MinMax { int32_t mn; int32_t mx; }`. Write the loop body as pure value selects (`x < mn ? x : mn`) — never as `if (x < mn) mn = x;` with early-outs or logging — so the compiler can lower it to conditional moves and vector min/max instructions. On random data an `if`-based scan mispredicts constantly; a select-based scan runs at memory speed.

Constraints: `1 <= n <= 10^6`. Single pass, O(1) extra space, no sorting, no early exit.

Example: `scanMinMax([7,-3,9,0], 4)` returns `{mn: -3, mx: 9}`. `scanMinMax([5], 1)` returns `{5, 5}`.

hint: Seed both `mn` and `mx` with `a[0]`, then fold the remaining elements — that handles `n == 1` and all-equal arrays for free.
hint: `mn = x < mn ? x : mn;` is a select on values (both sides are just values, no side effects) — compilers turn it into `cmov` scalar or `pminsd`/`pmaxsd` vector ops.
hint: Keep the loop body two selects and nothing else: no prints, no `if` statements, no function calls that could block vectorization.

```cpp
// starter
#include <cstdint>
#include <cstddef>
struct MinMax { int32_t mn; int32_t mx; };
MinMax scanMinMax(const int32_t* a, size_t n);
```

```cpp
struct MinMax { int32_t mn; int32_t mx; };

MinMax scanMinMax(const int32_t* a, size_t n) {
    int32_t mn = a[0];
    int32_t mx = a[0];
    for (size_t i = 1; i < n; ++i) {
        int32_t x = a[i];
        mn = x < mn ? x : mn;   // value select, not a control branch
        mx = x > mx ? x : mx;
    }
    return {mn, mx};
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
#include <climits>
//__USER__
int main() {
    int32_t single[] = {5};
    int32_t equal3[] = {3, 3, 3};
    int32_t up[]     = {1, 2, 3, 4, 5};
    int32_t down[]   = {5, 4, 3, 2, 1};
    int32_t wild[]   = {7, -3, INT_MAX, 0, INT_MIN, 42};
    struct { const int32_t* a; size_t n; int32_t mn, mx; } cases[] = {
        {single, 1, 5, 5},
        {equal3, 3, 3, 3},
        {up,     5, 1, 5},
        {down,   5, 1, 5},
        {wild,   6, INT_MIN, INT_MAX},
    };
    for (auto& c : cases) {
        MinMax r = scanMinMax(c.a, c.n);
        if (r.mn != c.mn || r.mx != c.mx) {
            std::printf("got {%d,%d} want {%d,%d}\n", r.mn, r.mx, c.mn, c.mx);
            return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** The distinction is control dependency versus data dependency. `if (x < mn) mn = x;` makes the instruction stream depend on the data: the branch predictor must guess, and on random input "is this a new minimum" is guessed wrong often early on (each mispredict flushes ~15–20 cycles of pipeline). `mn = x < mn ? x : mn;` computes both possibilities and selects — a `cmov` in scalar code — so there is nothing to predict. More importantly for throughput, a select-only loop body is exactly what auto-vectorizers need: clang and gcc compile this loop to `pminsd`/`pmaxsd` (or NEON `smin`/`smax`), processing 4–8 elements per cycle with the min/max lattice folded horizontally once at the end. Any side effect inside the conditional (a store, a call, an early `return`) would force real branches and kill the transform. Seeding with `a[0]` instead of `INT_MAX`/`INT_MIN` sentinels is a small robustness bonus: it works for any value range without assuming sentinels are unreachable.
