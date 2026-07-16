## challenge: Price ladder with incremental best tracking
tags: order-book, cache, hot-path
track: hft
difficulty: hard

The bid side of an order book, stored as a dense ladder: `qty[level]` for levels 0..1023, where a higher index is a better (higher) price. Implement `class BidLadder` with `void apply(int level, int64_t delta)` (add `delta` to the quantity at `level`; the caller guarantees quantity never goes negative), `int bestLevel() const` (highest level with nonzero quantity, `-1` if the book is empty), and `int64_t bestQty() const` (quantity at the best level, `0` if empty). The best must be maintained *incrementally*: `apply` is O(1) except when the current best empties, in which case it walks down to the next occupied level; the getters are O(1) reads.

Constraints: `0 <= level < 1024`; quantities and deltas fit `int64_t`; no scanning in the getters; `apply` must not scan when the update doesn't touch the best (an add below the best, or a partial reduction anywhere).

Example: `apply(500,+100); apply(510,+50);` → best is 510/50. `apply(510,-50);` empties the top → best falls back to 500/100. `apply(500,-100);` → empty book: `bestLevel() == -1`, `bestQty() == 0`.

hint: Keep `int best_` alongside the array. After writing the new quantity: if it's now positive and `level > best_`, the best simply improves to `level`.
hint: Only one case requires a scan: the quantity at `best_` just hit zero — walk `best_` downward while it points at an empty level (stopping at -1 for an empty book).
hint: Order the guard carefully: `while (best_ >= 0 && qty_[best_] == 0) --best_;` — check the bound before indexing.

```cpp
// starter
#include <cstdint>
class BidLadder {
public:
    void apply(int level, int64_t delta);  // qty[level] += delta; maintain best
    int bestLevel() const;                 // highest non-empty level, or -1
    int64_t bestQty() const;               // qty at best, or 0
};
```

```cpp
class BidLadder {
public:
    void apply(int level, int64_t delta) {
        int64_t q = qty_[level] + delta;
        qty_[level] = q;
        if (q > 0) {
            if (level > best_) best_ = level;          // improvement: O(1)
        } else if (level == best_) {
            while (best_ >= 0 && qty_[best_] == 0) --best_;   // retreat: walk down
        }
    }
    int bestLevel() const { return best_; }
    int64_t bestQty() const { return best_ >= 0 ? qty_[best_] : 0; }
private:
    static constexpr int kLevels = 1024;
    int64_t qty_[kLevels] = {};
    int best_ = -1;
};
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static bool expect(const BidLadder& b, int lvl, int64_t qty, const char* what) {
    if (b.bestLevel() != lvl || b.bestQty() != qty) {
        std::printf("%s: best=(%d,%lld) want (%d,%lld)\n", what,
                    b.bestLevel(), (long long)b.bestQty(), lvl, (long long)qty);
        return false;
    }
    return true;
}
int main() {
    BidLadder b;
    if (!expect(b, -1, 0, "fresh book")) return 1;
    b.apply(500, 100);
    if (!expect(b, 500, 100, "first add")) return 1;
    b.apply(510, 50);
    if (!expect(b, 510, 50, "better bid arrives")) return 1;
    b.apply(505, 75);
    if (!expect(b, 510, 50, "add below best keeps best")) return 1;
    b.apply(510, 25);
    if (!expect(b, 510, 75, "add at best updates qty")) return 1;
    b.apply(510, -25);
    if (!expect(b, 510, 50, "partial cancel keeps level")) return 1;
    b.apply(510, -50);
    if (!expect(b, 505, 75, "top empties, fall back one gap")) return 1;
    b.apply(505, -75);
    if (!expect(b, 500, 100, "fall back again")) return 1;
    b.apply(500, -100);
    if (!expect(b, -1, 0, "book empties")) return 1;
    b.apply(0, 5);
    if (!expect(b, 0, 5, "level 0 works")) return 1;
    b.apply(1023, 7);
    if (!expect(b, 1023, 7, "top level works")) return 1;
    b.apply(1023, -7);
    if (!expect(b, 0, 5, "long fall back 1023 -> 0")) return 1;
    b.apply(0, -5);
    if (!expect(b, -1, 0, "empty again")) return 1;
    b.apply(300, 10);
    if (!expect(b, 300, 10, "reuse after empty")) return 1;
    std::puts("PASS");
}
```

**Editorial:** This is the dense-array order book that fast equity feed handlers actually use: map price to a small integer index (price minus a base, divided by tick), keep quantities in a flat array, and carry the best as a cached index. The update taxonomy is the whole design: an add at or above the best and any partial reduction are O(1) — one array write plus a compare; the only structural event is "the best level just emptied," which triggers a downward walk to the next occupied level. That walk looks like the weak spot but rarely is: real markets cluster activity at and near the touch, so the gap below the vanished best is typically 1–2 ticks, and even a long walk streams through a contiguous `int64_t` array at 8 levels per cache line with the prefetcher ahead of you — compare a `std::map` book where *every* best-query chases red-black tree pointers across the heap. The previous challenge's bitmap is the standard upgrade when the walk must be bounded: keep a summary bitmap of occupied levels and the retreat becomes floor-log2 of two words, O(1) worst case. Note also what the guard order buys you: `best_ >= 0 && qty_[best_] == 0` short-circuits before indexing, so the empty-book case needs no sentinel level. Interviewers probe exactly these seams — the asymmetry of improve vs. retreat, and what data layout does to the "bad" case.
