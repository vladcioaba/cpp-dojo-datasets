## challenge: LIFO stack allocator
tags: stack-allocator, LIFO, arena
track: hft
difficulty: medium

A stack allocator is an arena that *can* free — as long as you free in reverse order of allocation (last-in, first-out), exactly like a call stack. Implement `StackAllocator` over a caller-supplied buffer: `allocate(n)` bumps the top up by `n` bytes and returns the old top (or `nullptr` if it won't fit); `deallocate(p)` rewinds the top back down to `p`, releasing that allocation and everything above it; `used()` reports bytes currently in use.

Constraints: `deallocate` must be called in LIFO order (pass the pointer returned by the matching `allocate`); operations are O(1). After freeing, the reclaimed space is handed out again by the next `allocate`.

Example: allocate `a`, `b`, `c` of 32 bytes each (`used() == 96`); `deallocate(c)` drops `used()` to 64 and the next 32-byte `allocate` returns `c`'s old address; `deallocate(a)` rewinds all the way to `used() == 0`.

hint: Keep a single `offset_` (the "top of stack"); `allocate` returns `base + offset_` and advances `offset_` by `n`.
hint: `deallocate(p)` rewinds the top to `p`'s offset: `offset_ = (char*)p - base`. That frees `p` and anything allocated after it in one shot.
hint: Reject an allocation that would push `offset_ + n` past the buffer size by returning `nullptr` and leaving the top unchanged.

```cpp
// starter
#include <cstddef>
class StackAllocator {
public:
    StackAllocator(void* buf, std::size_t size);
    void* allocate(std::size_t n);   // bump the top up; nullptr if it won't fit
    void  deallocate(void* p);       // rewind the top back to p (LIFO)
    std::size_t used() const;
};
```

```cpp
class StackAllocator {
    char* base_;
    std::size_t size_;
    std::size_t offset_ = 0;
public:
    StackAllocator(void* buf, std::size_t size)
        : base_(static_cast<char*>(buf)), size_(size) {}
    void* allocate(std::size_t n) {
        if (offset_ + n > size_) return nullptr;
        void* p = base_ + offset_;
        offset_ += n;
        return p;
    }
    void deallocate(void* p) {
        offset_ = static_cast<std::size_t>(static_cast<char*>(p) - base_);
    }
    std::size_t used() const { return offset_; }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
//__USER__
int main() {
    alignas(16) unsigned char buffer[128];
    StackAllocator sa(buffer, sizeof(buffer));
    auto U = [](void* p) { return reinterpret_cast<std::uintptr_t>(p); };

    void* a = sa.allocate(32);
    void* b = sa.allocate(32);
    void* c = sa.allocate(32);
    if (!a || !b || !c) { std::puts("allocate returned null"); return 1; }
    if (sa.used() != 96) { std::puts("used() should be 96"); return 1; }
    if (!(U(a) < U(b) && U(b) < U(c))) { std::puts("allocations not increasing"); return 1; }

    // No room for another 64 bytes.
    if (sa.allocate(64) != nullptr) { std::puts("should have exhausted"); return 1; }

    // LIFO free of c, then reuse its slot.
    sa.deallocate(c);
    if (sa.used() != 64) { std::puts("used() should be 64 after freeing c"); return 1; }
    void* c2 = sa.allocate(32);
    if (c2 != c) { std::puts("should reuse c's slot"); return 1; }

    // Rewind all the way back to the base.
    sa.deallocate(a);
    if (sa.used() != 0) { std::puts("used() should be 0 after freeing a"); return 1; }
    void* a2 = sa.allocate(8);
    if (a2 != a) { std::puts("should reuse from the base"); return 1; }

    std::puts("PASS");
}
```

**Editorial:** A stack allocator adds one capability to a bump arena: rewinding. Since the top only ever moves up on `allocate` and down on `deallocate`, freeing pointer `p` simply resets the top to `p`'s offset, which reclaims `p` and every allocation made after it. That is why deallocation must follow LIFO order — freeing out of order would leave the top inconsistent with live allocations. Like the arena it is O(1) with no per-block metadata, but the LIFO discipline lets you release nested scratch regions (e.g. temporaries within a single computation) without freeing the whole arena.
