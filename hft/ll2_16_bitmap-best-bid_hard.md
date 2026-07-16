## challenge: Best bid from a price-level bitmap
tags: order-book, bit-tricks, hot-path
track: hft
difficulty: hard

Fast order books track which price levels are occupied with a bitmap: one bit per level, and "best bid" is the highest set bit. Implement `class LevelBitmap` over 1024 price levels (index 0..1023, higher index = higher price = better bid): `void set(int idx)`, `void clear(int idx)`, and `int highest() const` returning the highest set index, or `-1` when empty. Store the levels in sixteen `uint64_t` words. Finding the top bit within a word must be O(log bits) — a fixed halving search, no per-bit loop, no `std::countl_zero`/builtins.

Constraints: `0 <= idx < 1024`; `set`/`clear` are O(1) (word index + mask); `highest` scans at most 16 words top-down and then does a 6-step floor-log2 inside the first non-empty word; `set`/`clear` are idempotent.

Example: after `set(500); set(510); set(3);` → `highest() == 510`; after `clear(510);` → `highest() == 500`; after clearing all → `highest() == -1`.

hint: Level `idx` lives in word `idx >> 6` at bit `idx & 63`: `words[idx >> 6] |= 1ull << (idx & 63)` to set, AND with the complement to clear.
hint: For `highest()`, walk words from 15 down to 0; the first non-zero word contains the answer: `(w << 6) + floorLog2(words[w])`.
hint: `floorLog2` by halving: `if (x >> 32) { n += 32; x >>= 32; }` then the same with 16, 8, 4, 2, 1 — six tests pin the top bit's index.

```cpp
// starter
#include <cstdint>
class LevelBitmap {
public:
    void set(int idx);      // mark price level idx occupied
    void clear(int idx);    // mark it empty
    int highest() const;    // highest occupied level, or -1 if none
};
```

```cpp
static int floorLog2(uint64_t x) {   // x != 0
    int n = 0;
    if (x >> 32) { n += 32; x >>= 32; }
    if (x >> 16) { n += 16; x >>= 16; }
    if (x >> 8)  { n += 8;  x >>= 8;  }
    if (x >> 4)  { n += 4;  x >>= 4;  }
    if (x >> 2)  { n += 2;  x >>= 2;  }
    if (x >> 1)  { n += 1; }
    return n;
}

class LevelBitmap {
public:
    void set(int idx)   { words_[idx >> 6] |=  (1ull << (idx & 63)); }
    void clear(int idx) { words_[idx >> 6] &= ~(1ull << (idx & 63)); }
    int highest() const {
        for (int w = kWords - 1; w >= 0; --w) {
            if (words_[w]) return (w << 6) + floorLog2(words_[w]);
        }
        return -1;
    }
private:
    static constexpr int kWords = 16;   // 16 * 64 = 1024 levels
    uint64_t words_[kWords] = {};
};
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
static bool expect(const LevelBitmap& b, int want, const char* what) {
    int got = b.highest();
    if (got != want) { std::printf("%s: highest()=%d want %d\n", what, got, want); return false; }
    return true;
}
int main() {
    LevelBitmap b;
    if (!expect(b, -1, "empty book")) return 1;
    b.set(0);
    if (!expect(b, 0, "only level 0")) return 1;
    b.set(63);
    if (!expect(b, 63, "top of word 0")) return 1;
    b.set(64);
    if (!expect(b, 64, "bottom of word 1")) return 1;
    b.set(500); b.set(510); b.set(3);
    if (!expect(b, 510, "several levels")) return 1;
    b.set(1023);
    if (!expect(b, 1023, "top level")) return 1;
    b.clear(1023);
    if (!expect(b, 510, "clear top falls back")) return 1;
    b.clear(510); b.clear(500);
    if (!expect(b, 64, "clear across words")) return 1;
    b.clear(64);
    if (!expect(b, 63, "word boundary 64 -> 63")) return 1;
    b.set(127); b.set(128);
    if (!expect(b, 128, "127/128 straddle")) return 1;
    b.clear(128);
    if (!expect(b, 127, "fall back to 127")) return 1;
    b.clear(127); b.clear(63); b.clear(3); b.clear(0);
    if (!expect(b, -1, "emptied book")) return 1;
    b.clear(5);   // clearing an already-clear level must be harmless
    if (!expect(b, -1, "idempotent clear")) return 1;
    b.set(200); b.set(200);   // idempotent set
    if (!expect(b, 200, "idempotent set")) return 1;
    std::puts("PASS");
}
```

**Editorial:** The bitmap is the classic answer to "how do you get O(1)-ish best-bid without a heap or a tree." A `std::map<price, level>` finds the best in O(log n) with pointer chases across scattered nodes — several cache misses each a hundred cycles. The bitmap is 128 bytes — two cache lines — that stay hot in L1 forever: `set`/`clear` are one shift, one mask, one RMW on a word (`idx >> 6` picks the word, `idx & 63` the bit — the same power-of-two indexing as ring buffers), and `highest()` is a top-down word scan plus a floor-log2. The halving `floorLog2` asks "is anything set in the upper half?" six times, each answer contributing one bit of the index — it's binary search over bit positions, and each test compiles to a shift, test, and conditional add (usually `cmov`). Hardware collapses the whole thing into one `bsr`/`lzcnt` (`std::countl_zero` in C++20), making the in-word part a single cycle. Production books add a *summary* word — bit `w` set iff `words[w] != 0` — so the scan itself becomes one more floor-log2 and `highest()` is truly constant-time; two levels of bitmap cover 4096 levels in three loads. Trade-off to name in an interview: bitmaps excel when the price universe is dense and bounded (equities near the touch); for sparse or unbounded ladders you fall back to trees or hashed levels.
