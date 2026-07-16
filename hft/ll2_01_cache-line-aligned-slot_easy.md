## challenge: One slot, one cache line
tags: cache, alignment, layout
track: hft
difficulty: easy

Define a market-data slot struct `MdSlot` that occupies exactly one 64-byte cache line and is 64-byte aligned, with fields in this exact order: `uint64_t seq`, `int64_t price`, `int32_t qty`, `uint32_t flags`. The compiler must place `seq` at offset 0, `price` at 8, `qty` at 16, `flags` at 20, with the rest of the line as tail padding. In an array of slots each element then starts on a fresh line, so a core reading slot `i` never drags slot `i+1`'s bytes into its cache — and a writer to slot `i+1` never invalidates the reader's line.

Constraints: `sizeof(MdSlot) == 64`, `alignof(MdSlot) == 64`, offsets exactly 0/8/16/20. No bit-fields; no manual `char pad[...]` member required.

Example: with `MdSlot slots[4];`, `&slots[1]` is exactly 64 bytes past `&slots[0]` and every element address is a multiple of 64.

hint: `alignas(64)` on the struct raises its alignment requirement, and `sizeof` is always a multiple of `alignof` — so the tail padding to 64 bytes comes for free.
hint: Order members largest-first (8, 8, 4, 4 bytes) so natural alignment yields offsets 0, 8, 16, 20 with no interior gaps.
hint: You never need to spell out the padding bytes: `struct alignas(64) MdSlot { ... };` forces `sizeof(MdSlot)` up to 64 by itself.

```cpp
// starter
#include <cstdint>
// Define MdSlot here: uint64_t seq, int64_t price, int32_t qty, uint32_t flags.
// It must occupy exactly one 64-byte cache line and be 64-byte aligned.
```

```cpp
struct alignas(64) MdSlot {
    uint64_t seq;     // offset 0
    int64_t  price;   // offset 8
    int32_t  qty;     // offset 16
    uint32_t flags;   // offset 20
    // bytes 24..63 are tail padding, supplied by alignas(64)
};
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
//__USER__
static_assert(sizeof(MdSlot) == 64, "must occupy exactly one cache line");
static_assert(alignof(MdSlot) == 64, "must be cache-line aligned");
static_assert(offsetof(MdSlot, seq) == 0, "seq at offset 0");
static_assert(offsetof(MdSlot, price) == 8, "price at offset 8");
static_assert(offsetof(MdSlot, qty) == 16, "qty at offset 16");
static_assert(offsetof(MdSlot, flags) == 20, "flags at offset 20");
int main() {
    static MdSlot slots[4] = {};
    for (int i = 0; i < 4; ++i) {
        if (reinterpret_cast<uintptr_t>(&slots[i]) % 64 != 0) {
            std::printf("slot %d is not 64-byte aligned\n", i); return 1;
        }
    }
    if (reinterpret_cast<uintptr_t>(&slots[1]) - reinterpret_cast<uintptr_t>(&slots[0]) != 64) {
        std::puts("array stride is not 64 bytes"); return 1;
    }
    slots[2].seq = 7; slots[2].price = -12345; slots[2].qty = 100; slots[2].flags = 3;
    if (slots[2].seq != 7 || slots[2].price != -12345 || slots[2].qty != 100 || slots[2].flags != 3) {
        std::puts("field round-trip failed"); return 1;
    }
    std::puts("PASS");
}
```

**Editorial:** The cache line (64 bytes on x86 and most ARM servers) is the unit of transfer and of coherence: when any byte in a line is written, every other core's copy of that whole line is invalidated. If two slots straddle one line, a writer updating slot 1 stalls a reader of slot 0 even though they touch disjoint data. `alignas(64)` fixes this structurally: it raises the struct's alignment to 64, and because the language guarantees `sizeof` is a multiple of `alignof` (arrays must tile), the compiler pads the struct to exactly 64 bytes — every array element owns a whole line. Ordering members largest-first (two 8-byte, then two 4-byte fields) packs them at offsets 0/8/16/20 with zero interior padding, keeping all the payload in the first 24 bytes so a single load of the line's start captures everything. `static_assert` + `offsetof` turn these layout assumptions into compile-time contracts — in a trading system you assert layout at build time, not discover it in a latency histogram.
