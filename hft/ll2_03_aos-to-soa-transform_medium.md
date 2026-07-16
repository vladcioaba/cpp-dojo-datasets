## challenge: AoS to SoA for the hot loop
tags: cache, simd, data-layout
track: hft
difficulty: medium

Quotes arrive as an array of structs: `struct Quote { int64_t price; int32_t qty; int32_t pad; };` (16 bytes each). A hot loop that only needs prices and quantities still drags every struct's full 16 bytes through the cache and can't vectorize cleanly. Split the data into parallel arrays. Implement `void toSoA(const Quote* q, size_t n, int64_t* prices, int32_t* qtys)` that scatters the AoS input into two dense arrays, and `int64_t notionalSum(const int64_t* prices, const int32_t* qtys, size_t n)` that returns the sum of `price * qty` over the SoA form in a single pass.

Constraints: `n` up to 10^6; every product and the total fit in `int64_t`; `prices`/`qtys` are caller-provided buffers of length `n`; `notionalSum` must be one linear pass with no extra allocation.

Example: quotes `{(100,2),(101,3),(-50,7)}` produce `prices == [100,101,-50]`, `qtys == [2,3,7]`, and `notionalSum == 100*2 + 101*3 + (-50)*7 == 153`.

hint: `toSoA` is a single loop copying `q[i].price` and `q[i].qty` into position `i` of each output array — the win is in the layout, not in clever code.
hint: In `notionalSum`, widen before multiplying: `prices[i]` is already `int64_t`, so `prices[i] * qtys[i]` is a 64-bit multiply — accumulate into an `int64_t`.
hint: Keep both loops free of branches and function calls so the compiler can unroll and vectorize them.

```cpp
// starter
#include <cstdint>
#include <cstddef>
struct Quote { int64_t price; int32_t qty; int32_t pad; };
void toSoA(const Quote* q, size_t n, int64_t* prices, int32_t* qtys);
int64_t notionalSum(const int64_t* prices, const int32_t* qtys, size_t n);
```

```cpp
struct Quote { int64_t price; int32_t qty; int32_t pad; };

void toSoA(const Quote* q, size_t n, int64_t* prices, int32_t* qtys) {
    for (size_t i = 0; i < n; ++i) {
        prices[i] = q[i].price;
        qtys[i]   = q[i].qty;
    }
}

int64_t notionalSum(const int64_t* prices, const int32_t* qtys, size_t n) {
    int64_t total = 0;
    for (size_t i = 0; i < n; ++i) {
        total += prices[i] * qtys[i];
    }
    return total;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
//__USER__
int main() {
    Quote q[5] = {
        {100, 2, 0}, {101, 3, 0}, {-50, 7, 0}, {99, 0, 0}, {1000000, 1000, 0},
    };
    int64_t prices[5] = {};
    int32_t qtys[5] = {};
    toSoA(q, 5, prices, qtys);
    for (size_t i = 0; i < 5; ++i) {
        if (prices[i] != q[i].price) { std::printf("prices[%zu] wrong\n", i); return 1; }
        if (qtys[i] != q[i].qty)     { std::printf("qtys[%zu] wrong\n", i); return 1; }
    }
    int64_t want = 0;
    for (size_t i = 0; i < 5; ++i) want += q[i].price * (int64_t)q[i].qty;
    int64_t got = notionalSum(prices, qtys, 5);
    if (got != want) { std::printf("notionalSum=%lld want %lld\n", (long long)got, (long long)want); return 1; }
    if (notionalSum(prices, qtys, 0) != 0) { std::puts("empty sum must be 0"); return 1; }
    toSoA(q, 0, prices, qtys);  // n == 0 must be a no-op
    std::puts("PASS");
}
```

**Editorial:** Layout decides bandwidth. In AoS, a loop that reads `price` and `qty` streams 16 bytes per quote but uses 12 — and worse, the fields it wants sit at a 16-byte stride, so a 64-byte cache line delivers only 4 quotes and the vectorizer must emit gather/shuffle sequences to pack values into SIMD lanes. In SoA, `prices[]` is a dense `int64_t` stream and `qtys[]` a dense `int32_t` stream: every byte fetched is used, hardware prefetchers see two perfectly sequential streams, and `notionalSum` compiles to straight-line SIMD (widen 4 qtys, multiply into 64-bit lanes, add) with no shuffles. The transform itself costs one pass, which is why real systems don't transform at all — feed handlers write SoA (or column-oriented) layouts from the start, and this exercise is the argument for that design. Rule of thumb: structure your storage around your loops, not around your nouns; AoS is for objects you handle one at a time, SoA is for fields you scan a million at a time.
