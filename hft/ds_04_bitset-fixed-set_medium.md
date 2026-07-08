## challenge: Bitset-backed fixed set
tags: allocation, bitset, set, bit-manipulation
track: hft
difficulty: medium

A set of small integers drawn from `[0, N)`, stored as a packed bitmap instead of a node-based container. Back it with an array of 64-bit words. Implement `insert(x)`, `bool contains(x) const`, `erase(x)`, and `count() const` (population count of live elements). Membership is a single bit test, insert/erase are single bit-set/clear operations, and the whole set of `N` possible elements fits in `ceil(N/64)` words.

Constraints: `N` is a compile-time constant, `1 <= N`. Elements satisfy `0 <= x < N`. No heap allocation. `insert` of a present element and `erase` of an absent element are both no-ops (idempotent).

Example: on `BitSet<130>` (3 words), `insert(1); insert(65); insert(129)` gives `count()==3` and `contains(65)==true`, `contains(2)==false`; `erase(65)` drops `count()` to 2; inserting `1` again keeps `count()` at 2.

hint: Element `x` lives at bit `x & 63` of word `x >> 6` — the word index is the high bits, the bit index is the low 6 bits.
hint: `insert` ORs in `1ull << (x & 63)`; `erase` ANDs with the complement of that mask; `contains` shifts the word right and tests the low bit.
hint: `count()` sums `__builtin_popcountll` over the words — the hardware `POPCNT` counts set bits in a word in one instruction.

```cpp
// starter
template <size_t N>
struct BitSet {
    static constexpr size_t W = (N + 63) / 64;   // number of 64-bit words
    uint64_t words_[W] = {};                      // all bits clear = empty set
    // implement insert / contains / erase / count
};
```

```cpp
void insert(size_t x) { words_[x >> 6] |= (uint64_t(1) << (x & 63)); }
bool contains(size_t x) const { return (words_[x >> 6] >> (x & 63)) & 1u; }
void erase(size_t x) { words_[x >> 6] &= ~(uint64_t(1) << (x & 63)); }
size_t count() const {
    size_t c = 0;
    for (size_t i = 0; i < W; ++i) c += (size_t)__builtin_popcountll(words_[i]);
    return c;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <size_t N>
struct BitSet {
    static constexpr size_t W = (N + 63) / 64;
    uint64_t words_[W] = {};
    //__USER__
};
int main() {
    BitSet<130> s;                              // spans 3 words
    if (s.count() != 0) { std::puts("empty count"); return 1; }
    if (s.contains(65)) { std::puts("empty contains"); return 1; }
    s.insert(1);                                // word 0
    s.insert(65);                               // word 1
    s.insert(129);                              // word 2, boundary
    if (s.count() != 3) { std::puts("count after inserts"); return 1; }
    if (!s.contains(1) || !s.contains(65) || !s.contains(129)) { std::puts("membership"); return 1; }
    if (s.contains(0) || s.contains(2) || s.contains(64) || s.contains(128)) { std::puts("false positives"); return 1; }
    s.insert(65);                               // idempotent
    if (s.count() != 3) { std::puts("insert must be idempotent"); return 1; }
    s.erase(65);
    if (s.contains(65) || s.count() != 2) { std::puts("erase failed"); return 1; }
    s.erase(64);                                // absent -> no-op
    if (s.count() != 2) { std::puts("erase of absent must be no-op"); return 1; }
    s.insert(63); s.insert(64);                 // straddle the word boundary
    if (!s.contains(63) || !s.contains(64) || s.count() != 4) { std::puts("word boundary"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A bitset represents a set over a bounded universe as raw bits, so a whole 128-element set is two machine words that live in a single cache line — versus `std::set<int>` (a red-black tree: one heap node with three pointers and a color bit per element, scattered across memory) or even `std::unordered_set` (buckets plus nodes). Insert, erase, and membership become single ALU ops with no branches and no pointer chasing, and `count()` rides the hardware `POPCNT`. When your keys are dense small integers — order-book price levels, active session ids, a free/used mask — the bitset is both the smallest and the fastest representation, with zero allocation.
