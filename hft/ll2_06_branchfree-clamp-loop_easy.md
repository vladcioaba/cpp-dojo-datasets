## challenge: Branch-free clamp over an array
tags: simd, branch-prediction, hot-path
track: hft
difficulty: easy

Risk checks clamp order sizes and prices into `[lo, hi]` for millions of values per second. Implement `void clampAll(int32_t* a, size_t n, int32_t lo, int32_t hi)` that clamps every element in place — written branch-free: the loop body must be two value selects (`x < lo ? lo : x`, then `x > hi ? hi : x`), not `if`/`else if` statements. Selects lower to min/max instructions and let the whole loop vectorize; branches on unpredictable data flush the pipeline element after element.

Constraints: `lo <= hi`; `0 <= n <= 10^6`; in-place, single pass, no early exit, no `if` statements in the loop body.

Example: with `lo = -5, hi = 5`: `[-100, -5, 0, 7, 5]` becomes `[-5, -5, 0, 5, 5]`. With `lo == hi == 0` every element becomes `0`.

hint: Load once, select twice, store once: `int32_t x = a[i]; x = x < lo ? lo : x; x = x > hi ? hi : x; a[i] = x;`
hint: Apply the lower bound first, then the upper — with `lo <= hi` the order gives the correct result for every input.
hint: Both selects have plain values on both arms (no side effects), which is exactly the shape compilers turn into `pmaxsd`/`pminsd` over 4–8 lanes at a time.

```cpp
// starter
#include <cstdint>
#include <cstddef>
void clampAll(int32_t* a, size_t n, int32_t lo, int32_t hi);
```

```cpp
void clampAll(int32_t* a, size_t n, int32_t lo, int32_t hi) {
    for (size_t i = 0; i < n; ++i) {
        int32_t x = a[i];
        x = x < lo ? lo : x;   // max(x, lo)
        x = x > hi ? hi : x;   // min(x, hi)
        a[i] = x;
    }
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
#include <climits>
//__USER__
static void refClamp(const int32_t* in, int32_t* out, size_t n, int32_t lo, int32_t hi) {
    for (size_t i = 0; i < n; ++i) {
        int32_t x = in[i];
        if (x < lo) x = lo;
        if (x > hi) x = hi;
        out[i] = x;
    }
}
static bool runCase(const int32_t* in, size_t n, int32_t lo, int32_t hi) {
    int32_t work[16], want[16];
    for (size_t i = 0; i < n; ++i) work[i] = in[i];
    refClamp(in, want, n, lo, hi);
    clampAll(work, n, lo, hi);
    for (size_t i = 0; i < n; ++i) {
        if (work[i] != want[i]) {
            std::printf("i=%zu got %d want %d (lo=%d hi=%d)\n", i, work[i], want[i], lo, hi);
            return false;
        }
    }
    return true;
}
int main() {
    int32_t data[] = {INT_MIN, -100, -5, -1, 0, 1, 5, 7, 100, INT_MAX};
    if (!runCase(data, 10, -5, 5)) return 1;
    if (!runCase(data, 10, 0, 0)) return 1;                 // lo == hi
    if (!runCase(data, 10, INT_MIN, INT_MAX)) return 1;     // identity
    if (!runCase(data, 0, -5, 5)) return 1;                 // empty
    if (!runCase(data, 1, 3, 9)) return 1;                  // single element
    std::puts("PASS");
}
```

**Editorial:** Clamping is the canonical "unpredictable branch" workload: whether an element is below, inside, or above the band is data-dependent, so an `if (x < lo) ... else if (x > hi) ...` loop mispredicts on mixed input and each mispredict costs ~15–20 cycles — often more than the useful work in the entire iteration. Writing the body as two selects removes control flow entirely: `x < lo ? lo : x` is `max(x, lo)` and `x > hi ? hi : x` is `min(x, hi)`, and both gcc and clang lower the loop to packed `pmaxsd`/`pminsd` (or NEON `smax`/`smin`), clamping 4–8 elements per instruction with zero mispredict exposure. The order matters only for correctness bookkeeping: applying `max` with `lo` first, then `min` with `hi`, is correct whenever `lo <= hi` (this is also exactly how `std::clamp` composes). The general lesson: on hot paths, convert *control* dependencies into *data* dependencies whenever both arms are cheap pure values — the CPU is far better at streaming computation than at guessing your data.
