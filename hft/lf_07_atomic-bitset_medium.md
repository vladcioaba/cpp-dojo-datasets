## challenge: Atomic bitset test-and-set
tags: atomics, bitset, test-and-set
track: hft
difficulty: medium

A fixed-size bitset that many threads probe to **claim slots exactly once** — think a pool of `Bits` order buffers where `test_and_set(i)` returns `false` for the caller that won slot `i` and `true` for everyone who arrives after. Back it with an array of `std::atomic<uint64_t>` words (`W = ceil(Bits/64)`). Implement `bool test_and_set(size_t i)` (atomically set bit `i`, return its *previous* value), `bool test(size_t i) const`, and `void reset(size_t i)`. Bit `i` lives in word `i >> 6` at position `i & 63`. Single-threaded correctness; the harness checks the claim-once idiom and cross-word independence.

Constraints: use `fetch_or` / `fetch_and` on the word — never a load-modify-store, which would race with a claim on a neighbouring bit in the same word.

Example: `test_and_set(65)` on a clear bitset returns `false` (you won it) and sets the bit; a second `test_and_set(65)` returns `true`; setting bit 129 leaves bit 65 untouched (different word).

hint: Setting one bit atomically is `fetch_or(1ull << (i & 63))` on `words_[i >> 6]`; the return value is the whole old word, so mask it to recover the old bit.
hint: `reset` clears a bit with `fetch_and(~mask)` — again a single atomic RMW, not read-modify-write.
hint: `test_and_set` on shared state needs `acq_rel` so the claim both publishes and observes; a pure `test` load can be `acquire`.

```cpp
// starter
#include <atomic>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <size_t Bits>
struct AtomicBitset {
    static constexpr size_t W = (Bits + 63) / 64;
    std::atomic<std::uint64_t> words_[W];
    AtomicBitset() { for (size_t i = 0; i < W; ++i) words_[i].store(0, std::memory_order_relaxed); }
    bool test_and_set(size_t i);   // set bit i, return its previous value
    bool test(size_t i) const;
    void reset(size_t i);
};
```

```cpp
bool test_and_set(size_t i) {
    std::uint64_t mask = std::uint64_t(1) << (i & 63);
    std::uint64_t prev = words_[i >> 6].fetch_or(mask, std::memory_order_acq_rel);
    return (prev & mask) != 0;
}
bool test(size_t i) const {
    std::uint64_t mask = std::uint64_t(1) << (i & 63);
    return (words_[i >> 6].load(std::memory_order_acquire) & mask) != 0;
}
void reset(size_t i) {
    std::uint64_t mask = std::uint64_t(1) << (i & 63);
    words_[i >> 6].fetch_and(~mask, std::memory_order_release);
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <size_t Bits>
struct AtomicBitset {
    static constexpr size_t W = (Bits + 63) / 64;
    std::atomic<std::uint64_t> words_[W];
    AtomicBitset() { for (size_t i = 0; i < W; ++i) words_[i].store(0, std::memory_order_relaxed); }
    //__USER__
};
int main() {
    AtomicBitset<130> bs;
    for (size_t i = 0; i < 130; ++i) if (bs.test(i)) { std::puts("must start clear"); return 1; }
    if (bs.test_and_set(65)) { std::puts("first t&s must report was-clear"); return 1; }
    if (!bs.test(65)) { std::puts("bit must be set"); return 1; }
    if (!bs.test_and_set(65)) { std::puts("second t&s must report was-set"); return 1; }
    if (bs.test(1) || bs.test(64) || bs.test(129)) { std::puts("no cross-word bleed"); return 1; }
    if (bs.test_and_set(129)) { std::puts("129 was clear"); return 1; }
    if (!bs.test(129) || !bs.test(65)) { std::puts("both must be set"); return 1; }
    bs.reset(65);
    if (bs.test(65)) { std::puts("reset failed"); return 1; }
    if (!bs.test(129)) { std::puts("reset must not bleed"); return 1; }
    int claimed = 0;
    for (int r = 0; r < 3; ++r)
        for (size_t i = 0; i < 130; ++i)
            if (!bs.test_and_set(i)) ++claimed;   // count first-time wins
    if (claimed != 130 - 1) { std::puts("each slot claimed once"); return 1; } // 129 stays set; 65 re-won
    std::puts("PASS");
}
```

**Editorial:** The whole reason to store the bitset as an array of atomic words rather than a plain `bool[]` is that neighbouring bits share a word: two threads claiming bit 3 and bit 5 both write the same 64-bit cell. A naive `word = word | mask` (load, OR, store) would let one thread's store clobber the other's, silently un-setting a claim. `fetch_or` performs the OR atomically inside the memory system, so every bit's set is durable regardless of concurrent sets to its word-mates — and it conveniently returns the *entire* prior word, so masking recovers whether *this* bit was already set. That old-bit value is what makes `test_and_set` a one-shot claim primitive: exactly one caller sees `false`. Minimal-correct ordering: a claim that guards other work needs `acq_rel` (release to publish what the winner does with the slot, acquire to see prior owners' effects); a read-only `test` needs only `acquire`, and `reset` needs `release`. If you never synchronize other memory through the bitset, all of these collapse to `relaxed` — the atomicity of `fetch_or`, not the ordering, is what prevents lost bits.
