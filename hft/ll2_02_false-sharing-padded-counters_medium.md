## challenge: False-sharing-free counter array
tags: cache, false-sharing, layout
track: hft
difficulty: medium

Per-thread statistics counters that sit next to each other in memory destroy each other's cache lines: every increment by one thread invalidates the line in every other core. Fix it structurally. Implement `PaddedCounter` — a `uint64_t value` that owns a full 64-byte cache line — and `Counters`, a fixed bank of 8 of them with `void add(size_t i, uint64_t d)`, `uint64_t get(size_t i) const`, and `uint64_t total() const`. Adjacent counters must never share a line.

Constraints: `sizeof(PaddedCounter) == 64` and `alignof(PaddedCounter) == 64`; `0 <= i < 8`; counters start at zero; `add`/`get` are O(1) array indexing (no map, no hash).

Example: `Counters c; c.add(0,5); c.add(7,2); c.add(0,1);` then `c.get(0) == 6`, `c.get(7) == 2`, `c.total() == 8`. In `PaddedCounter pair[2];`, `&pair[1].value` is exactly 64 bytes past `&pair[0].value`.

hint: `alignas(64)` on the struct does two jobs at once: it aligns the first instance to a line boundary and rounds `sizeof` up to 64, so array elements land on distinct lines.
hint: A manual `char pad[56]` after the value also works, but the invariant you actually need is `sizeof == 64` — let the compiler do the padding.
hint: `Counters` is just `PaddedCounter slots_[8]` behind bounds-free O(1) indexing; `total()` walks the 8 slots.

```cpp
// starter
#include <cstdint>
#include <cstddef>
// struct PaddedCounter { uint64_t value; ... };  // one full cache line
// class Counters {  // bank of 8 padded counters, all starting at 0
//     void add(size_t i, uint64_t d);
//     uint64_t get(size_t i) const;
//     uint64_t total() const;
// };
```

```cpp
struct alignas(64) PaddedCounter {
    uint64_t value = 0;
};

class Counters {
public:
    void add(size_t i, uint64_t d) { slots_[i].value += d; }
    uint64_t get(size_t i) const { return slots_[i].value; }
    uint64_t total() const {
        uint64_t t = 0;
        for (const PaddedCounter& s : slots_) t += s.value;
        return t;
    }
private:
    PaddedCounter slots_[8];
};
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
//__USER__
static_assert(sizeof(PaddedCounter) == 64, "counter must own a full cache line");
static_assert(alignof(PaddedCounter) == 64, "counter must be line aligned");
int main() {
    PaddedCounter pair[2];
    uintptr_t d = reinterpret_cast<uintptr_t>(&pair[1].value)
                - reinterpret_cast<uintptr_t>(&pair[0].value);
    if (d != 64) { std::printf("adjacent values %u bytes apart, want 64\n", (unsigned)d); return 1; }
    if (reinterpret_cast<uintptr_t>(&pair[0]) % 64 != 0) { std::puts("not line aligned"); return 1; }

    Counters c;
    for (size_t i = 0; i < 8; ++i) {
        if (c.get(i) != 0) { std::printf("counter %zu not zero-initialized\n", i); return 1; }
    }
    c.add(0, 5); c.add(7, 2); c.add(0, 1); c.add(3, 100);
    if (c.get(0) != 6)  { std::puts("get(0) wrong"); return 1; }
    if (c.get(7) != 2)  { std::puts("get(7) wrong"); return 1; }
    if (c.get(3) != 100){ std::puts("get(3) wrong"); return 1; }
    if (c.get(1) != 0)  { std::puts("get(1) should be untouched"); return 1; }
    if (c.total() != 108) { std::printf("total=%llu want 108\n", (unsigned long long)c.total()); return 1; }
    std::puts("PASS");
}
```

**Editorial:** False sharing is coherence traffic without actual sharing: eight plain `uint64_t` counters fit in one 64-byte line, so when thread A increments counter 0, the MESI protocol invalidates the line in the core running thread B, which was only ever touching counter 1. Each increment becomes a cross-core round trip (~40–100 ns) instead of an L1 hit (~1 ns) — a 50–100x slowdown that profiles as "memory bound" with no obvious culprit. The fix costs only space: give each counter its own line. `alignas(64)` guarantees both the alignment of the first element and, because `sizeof` must be a multiple of `alignof`, the 64-byte stride of every element — no fragile hand-counted `char pad[56]`. The `static_assert`s make the layout a compile-time contract so a colleague adding a field can't silently reintroduce sharing. C++17's `std::hardware_destructive_interference_size` names the constant, though many shops pin 64 explicitly (and use 128 on CPUs that prefetch line pairs). The same pattern protects SPSC queue head/tail indices and per-core sequence numbers.
