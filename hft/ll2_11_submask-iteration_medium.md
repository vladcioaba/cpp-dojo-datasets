## challenge: Enumerate every submask
tags: bit-tricks, hot-path
track: hft
difficulty: medium

A bitmask encodes a set — venues to route to, flags on an order, feature combinations. Sometimes you must visit every *subset* of that set. Implement `uint64_t foldSubmasks(uint32_t mask)` that visits every submask `s` of `mask` (every `s` with `s & mask == s`, including `mask` itself and `0`) exactly once, in decreasing numeric order starting from `mask` and ending at `0`, folding each into an accumulator as `acc = acc * 3 + s` (with `acc` starting at 0, arithmetic in `uint64_t`), and returns the final `acc`.

Constraints: must run in O(2^k) where `k = popcount(mask)` — scanning all values from `mask` down to 0 and testing each is disallowed. For the tests, `popcount(mask) <= 8`.

Example: `foldSubmasks(0b101)` visits `5, 4, 1, 0` in that order: acc = 5, then 5*3+4 = 19, then 19*3+1 = 58, then 58*3+0 = 174 — returns `174`. `foldSubmasks(0) == 0` (visits just `0`).

hint: The classic enumeration: start at `s = mask`, and step with `s = (s - 1) & mask` — the decrement borrows through the trailing zeros, the AND throws away everything outside the mask.
hint: That step visits all submasks in strictly decreasing numeric order and reaches `0` last; loop `while (true)`, fold, and break *after* processing `s == 0` (stepping from 0 would wrap back to `mask`).
hint: Fold first, test for zero second: `acc = acc*3 + s; if (s == 0) break; s = (s - 1) & mask;`

```cpp
// starter
#include <cstdint>
uint64_t foldSubmasks(uint32_t mask);
```

```cpp
uint64_t foldSubmasks(uint32_t mask) {
    uint64_t acc = 0;
    uint32_t s = mask;
    while (true) {
        acc = acc * 3 + s;
        if (s == 0) break;
        s = (s - 1) & mask;
    }
    return acc;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static uint64_t reference(uint32_t mask) {
    // Brute force: scan every value from mask down to 0, keep those inside mask.
    uint64_t acc = 0;
    for (uint64_t v = mask; ; --v) {
        if ((v & mask) == v) acc = acc * 3 + v;
        if (v == 0) break;
    }
    return acc;
}
int main() {
    if (foldSubmasks(0u) != 0ull) { std::puts("mask 0 must fold to 0"); return 1; }
    if (foldSubmasks(1u) != 3ull) { std::puts("mask 1 must fold to 3"); return 1; }      // 1 then 0
    if (foldSubmasks(0b101u) != 174ull) { std::puts("mask 0b101 must fold to 174"); return 1; }
    uint32_t masks[] = {0b11u, 0b1101u, 0xFFu, 0x92u, 0b110100000000u};
    for (uint32_t m : masks) {
        uint64_t got = foldSubmasks(m);
        uint64_t want = reference(m);
        if (got != want) {
            std::printf("foldSubmasks(0x%x)=%llu want %llu\n",
                        m, (unsigned long long)got, (unsigned long long)want);
            return 1;
        }
    }
    // spread-bits case: brute reference would scan 2^31 values, so the
    // expected fold (submasks 0x80000001, 0x80000000, 1, 0) is precomputed —
    // a submask walk visits 4 values here, the naive scan 2 billion
    if (foldSubmasks(0x80000001u) != 77309411358ull) {
        std::puts("foldSubmasks(0x80000001) wrong"); return 1;
    }
    std::puts("PASS");
}
```

**Editorial:** The step `s = (s - 1) & mask` is subset decrement: subtracting 1 clears the lowest set bit of `s` and sets every bit below it, and ANDing with `mask` keeps only the positions that belong to the set — the net effect is "the next smaller integer that is still a submask." Because every step strictly decreases `s` and never skips a valid submask (any submask between `s-1` and the result would survive the AND), the walk visits all `2^k` subsets of a k-bit mask in decreasing numeric order, ending at 0. The subtle trap is termination: from `s == 0` the step wraps to `mask` and loops forever, so you process 0 first, *then* break — the do-while-style structure in the solution. Cost is exactly `2^k` iterations regardless of how the k bits are spread across the word (the harness's `0x80000001` case), versus `mask+1` iterations for the naive scan — for 8 bits spread over a 32-bit word that's 256 versus 2 billion. This enumeration underpins subset-sum DP ("sum over submasks", total work 3^k across all masks), routing over venue combinations, and exhaustive small-set searches where the set is already a bitmask in a register.
