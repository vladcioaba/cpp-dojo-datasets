## challenge: Bump (arena) allocator
tags: arena, bump-allocator, alignment
track: hft
difficulty: medium

The fastest allocator is a pointer you bump forward. Implement `Arena` over a caller-supplied buffer: `allocate(n, align)` carves `n` bytes at the next `align`-aligned offset and returns the pointer (or `nullptr` if the buffer can't fit it), and `reset()` reclaims everything at once by rewinding the offset to zero. No per-object free — that is the whole point.

Constraints: `align` is a power of two; do not read or write past the buffer; allocation is O(1). The returned pointer must satisfy `reinterpret_cast<uintptr_t>(p) % align == 0`.

Example: over a 64-byte-aligned 256-byte buffer, `allocate(10, 16)` returns the base; a following `allocate(64, 32)` returns the next 32-aligned slot after those 10 bytes; a request larger than the remaining space returns `nullptr`; after `reset()` the next allocation reuses the base again.

hint: Track a single `offset_` into the buffer; align the *current* address (base + offset) up to `align` before handing out `n` bytes.
hint: Compute the aligned address with the mask trick `(addr + align - 1) & ~(align - 1)`, then check the resulting end offset against the buffer size *before* committing.
hint: `reset()` is just `offset_ = 0` — there are no destructors to run and no per-block bookkeeping.

```cpp
// starter
#include <cstddef>
class Arena {
public:
    Arena(void* buf, std::size_t size);
    void* allocate(std::size_t n, std::size_t align);
    void reset();
};
```

```cpp
class Arena {
    char* base_;
    std::size_t size_;
    std::size_t offset_ = 0;
public:
    Arena(void* buf, std::size_t size)
        : base_(static_cast<char*>(buf)), size_(size) {}

    void* allocate(std::size_t n, std::size_t align) {
        std::uintptr_t cur     = reinterpret_cast<std::uintptr_t>(base_) + offset_;
        std::uintptr_t aligned = (cur + (align - 1)) & ~(align - 1);
        std::size_t new_offset =
            static_cast<std::size_t>(aligned - reinterpret_cast<std::uintptr_t>(base_)) + n;
        if (new_offset > size_) return nullptr;   // would overrun the buffer
        offset_ = new_offset;
        return reinterpret_cast<void*>(aligned);
    }

    void reset() { offset_ = 0; }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
//__USER__
int main() {
    alignas(64) unsigned char buffer[256];
    Arena a(buffer, sizeof(buffer));

    void* p1 = a.allocate(10, 16);
    if (!p1 || reinterpret_cast<std::uintptr_t>(p1) % 16 != 0) { std::puts("p1 alignment"); return 1; }

    void* p2 = a.allocate(1, 1);
    if (!p2) { std::puts("p2 null"); return 1; }
    if (reinterpret_cast<std::uintptr_t>(p2) < reinterpret_cast<std::uintptr_t>(p1) + 10) { std::puts("p2 overlaps p1"); return 1; }

    void* p3 = a.allocate(64, 32);
    if (!p3 || reinterpret_cast<std::uintptr_t>(p3) % 32 != 0) { std::puts("p3 alignment"); return 1; }

    // No room left for a giant request.
    if (a.allocate(1024, 8) != nullptr) { std::puts("should have exhausted"); return 1; }

    // reset() reclaims everything; the next allocation reuses the base.
    a.reset();
    void* p4 = a.allocate(10, 16);
    if (p4 != p1) { std::puts("reset should reuse from the start"); return 1; }

    std::puts("PASS");
}
```

**Editorial:** An arena keeps a single monotonically increasing offset. Each `allocate` aligns the current cursor up to the requested boundary (mask trick, since `align` is a power of two), checks that the resulting end stays within the buffer, and only then commits the new offset — so a failed allocation leaves state untouched and simply returns `nullptr`. There is no free list and no per-object metadata, which is why allocation is a handful of instructions and cache-friendly. The trade-off: you cannot free individual objects; you reclaim the whole region at once with `reset()`, ideal for per-tick or per-request scratch memory in a trading loop.
