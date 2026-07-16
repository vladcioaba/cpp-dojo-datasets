## challenge: Aligned bump allocator
tags: memory, allocator, hot-path
track: hft
difficulty: medium

`malloc` takes locks, maintains free lists, and can syscall — none of which belongs on a hot path. The trading-system workhorse is the bump (arena) allocator: allocation is a pointer add, deallocation is wholesale `reset()` after the tick. Implement `class Arena` over a caller-provided buffer: `Arena(unsigned char* buf, size_t cap)`, `void* allocate(size_t size, size_t align)` (align is a power of two; returns a pointer aligned to `align`, or `nullptr` if it doesn't fit — *without* consuming any space on failure), `void reset()`, and `size_t used() const` (bytes consumed including alignment padding).

Constraints: the caller guarantees `buf` is aligned at least as strictly as any `align` it will request (the harness passes a 64-byte-aligned buffer and aligns up to 64). `allocate` is O(1): align the offset up with bit arithmetic, no loops, no searching. A failed allocation must leave the arena unchanged. An allocation may exactly reach `cap`.

Example: with a 256-byte arena: `allocate(1,1)` returns `buf` (used = 1); `allocate(8,8)` returns `buf+8` (offset bumped from 1 to 8; used = 16); `allocate(1000,1)` returns `nullptr` (used still 16); `reset()` makes `used() == 0` and the next allocation returns `buf` again.

hint: Align the *offset*, not the pointer: `aligned = (off + (align - 1)) & ~(align - 1)` rounds up to the next multiple of a power of two; since `buf` itself is aligned, `buf + aligned` is too.
hint: Check for space without overflow: test `aligned > cap` first, then `size > cap - aligned` — the naive `aligned + size > cap` can wrap for huge `size`.
hint: Only commit state (`off = aligned + size`) after both checks pass — failure must be side-effect-free.

```cpp
// starter
#include <cstddef>
class Arena {
public:
    Arena(unsigned char* buf, size_t cap);
    void* allocate(size_t size, size_t align);  // aligned pointer, or nullptr if it doesn't fit
    void reset();                               // free everything at once
    size_t used() const;                        // bytes consumed, padding included
};
```

```cpp
class Arena {
public:
    Arena(unsigned char* buf, size_t cap) : buf_(buf), cap_(cap), off_(0) {}

    void* allocate(size_t size, size_t align) {
        size_t aligned = (off_ + (align - 1)) & ~(align - 1);
        if (aligned > cap_ || size > cap_ - aligned) return nullptr;  // overflow-safe
        void* p = buf_ + aligned;
        off_ = aligned + size;
        return p;
    }

    void reset() { off_ = 0; }
    size_t used() const { return off_; }

private:
    unsigned char* buf_;
    size_t cap_;
    size_t off_;
};
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
//__USER__
alignas(64) static unsigned char buf[256];
int main() {
    Arena a(buf, sizeof(buf));
    if (a.used() != 0) { std::puts("fresh arena must be empty"); return 1; }

    void* p1 = a.allocate(1, 1);
    if (p1 != buf) { std::puts("first alloc must start at buf"); return 1; }
    if (a.used() != 1) { std::puts("used after 1-byte alloc must be 1"); return 1; }

    void* p2 = a.allocate(8, 8);
    if (p2 != buf + 8) { std::puts("8-aligned alloc must land at offset 8"); return 1; }
    if (a.used() != 16) { std::puts("used must include alignment padding"); return 1; }

    void* p3 = a.allocate(4, 16);
    if (p3 != buf + 16) { std::puts("16-aligned alloc must land at offset 16"); return 1; }
    if (reinterpret_cast<uintptr_t>(p3) % 16 != 0) { std::puts("pointer not 16-aligned"); return 1; }
    if (a.used() != 20) { std::puts("used must be 20"); return 1; }

    if (a.allocate(1000, 1) != nullptr) { std::puts("oversized alloc must fail"); return 1; }
    if (a.used() != 20) { std::puts("failed alloc must not consume space"); return 1; }

    void* p4 = a.allocate(236, 4);   // 20 is 4-aligned; 20 + 236 == 256: exact fit allowed
    if (p4 != buf + 20) { std::puts("exact-fit alloc must succeed"); return 1; }
    if (a.used() != 256) { std::puts("arena must now be full"); return 1; }
    if (a.allocate(1, 1) != nullptr) { std::puts("full arena must refuse"); return 1; }

    a.reset();
    if (a.used() != 0) { std::puts("reset must empty the arena"); return 1; }
    void* p5 = a.allocate(64, 64);
    if (p5 != buf) { std::puts("post-reset alloc must start at buf again"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A bump allocator is the fastest allocation scheme that exists: round the offset up, compare, add — three ALU ops, no metadata per block, no locks, no free lists, and consecutive allocations are contiguous in memory (great for cache locality of related objects). Its contract is what makes it fast: you cannot free individual objects, only `reset()` the whole arena — which fits event-driven systems perfectly, where everything allocated while processing one packet dies when the packet is done ("per-tick arena"). The align-up idiom `(off + align-1) & ~(align-1)` works only for power-of-two `align` — the mask clears the low bits — and aligning the *offset* is valid because the base buffer is at least as aligned as any request, so alignment is preserved by addition. Two production-grade details the tests enforce: the space check must be overflow-safe (`size > cap - aligned` after checking `aligned <= cap`, since `aligned + size` can wrap `size_t`), and failure must be transactional — a `nullptr` return that also corrupted `off_` would poison every later allocation. `std::pmr::monotonic_buffer_resource` is this exact idea productized; writing one from scratch is a standing interview question because it exposes whether you understand alignment, overflow, and ownership at once.
