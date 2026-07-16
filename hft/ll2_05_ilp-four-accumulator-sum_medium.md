## challenge: Sum with four independent accumulators
tags: ilp, simd, hot-path
track: hft
difficulty: medium

A naive `total += a[i]` loop is a single serial dependency chain: each add must wait for the previous one, so the CPU's multiple ALUs sit idle. Implement `int64_t sum4(const int32_t* a, size_t n)` that sums the array using four independent 64-bit accumulators — `s0..s3`, each fed by every 4th element — then combines them at the end, with a scalar tail loop for the leftover `n % 4` elements. Same answer, but the four chains run in parallel through the pipeline.

Constraints: `0 <= n <= 10^6`; elements are arbitrary `int32_t`; the true sum fits in `int64_t` (accumulate in 64-bit — a 32-bit accumulator would overflow). Exactly one pass over the data.

Example: `sum4([3,-1,4,-1,5,9,-2], 7) == 17`. `sum4([], 0) == 0`.

hint: Main loop strides by 4 while `i + 4 <= n`: `s0 += a[i]; s1 += a[i+1]; s2 += a[i+2]; s3 += a[i+3];` — no accumulator reads another's result inside the loop.
hint: Finish leftovers with a plain loop into `s0`, then return `(s0 + s1) + (s2 + s3)` — a balanced reduction tree.
hint: Each `int32_t` element must be widened into an `int64_t` accumulator; declare `s0..s3` as `int64_t` and the conversion is implicit.

```cpp
// starter
#include <cstdint>
#include <cstddef>
int64_t sum4(const int32_t* a, size_t n);
```

```cpp
int64_t sum4(const int32_t* a, size_t n) {
    int64_t s0 = 0, s1 = 0, s2 = 0, s3 = 0;
    size_t i = 0;
    for (; i + 4 <= n; i += 4) {
        s0 += a[i];
        s1 += a[i + 1];
        s2 += a[i + 2];
        s3 += a[i + 3];
    }
    for (; i < n; ++i) s0 += a[i];   // tail: 0..3 leftover elements
    return (s0 + s1) + (s2 + s3);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
//__USER__
static int32_t big[100000];
int main() {
    int32_t a[] = {3, -1, 4, -1, 5, 9, -2};
    for (size_t n = 0; n <= 7; ++n) {   // every tail length 0..3, plus n==0
        int64_t want = 0;
        for (size_t i = 0; i < n; ++i) want += a[i];
        int64_t got = sum4(a, n);
        if (got != want) { std::printf("n=%zu got %lld want %lld\n", n, (long long)got, (long long)want); return 1; }
    }
    for (int i = 0; i < 100000; ++i) big[i] = (i % 2) ? 2000000000 : -1000000000;
    int64_t want = 0;
    for (int i = 0; i < 100000; ++i) want += big[i];   // 5e13: overflows int32 by far
    int64_t got = sum4(big, 100000);
    if (got != want) { std::printf("big got %lld want %lld\n", (long long)got, (long long)want); return 1; }
    std::puts("PASS");
}
```

**Editorial:** This is instruction-level parallelism made explicit. An add has ~1 cycle latency, but a modern core can *issue* 3–4 adds per cycle — the naive loop can't exploit that because `total += a[i]` forms one serial chain: iteration i+1's add consumes iteration i's result. Four accumulators create four independent chains, so the scheduler keeps 4 adds in flight and throughput approaches the load bandwidth limit rather than the add-latency limit. This is exactly the transform auto-vectorizers and `-ffast-math` reassociation perform for floats; for integers the compiler may already do it at `-O2`/`-O3`, but interviewers want you to know *why* it works, because the same reasoning applies where compilers can't help: chained FP sums (reassociation changes rounding, so the compiler must preserve your chain), latency-bound hash loops, and pointer chases. The reduction tree `(s0+s1)+(s2+s3)` finishes in 2 dependent steps instead of 3. Widening to `int64_t` per-accumulator also removes the overflow trap: 100k elements of ±2·10^9 exceed `int32_t` range a thousandfold.
