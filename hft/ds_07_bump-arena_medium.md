## challenge: Monotonic bump / arena allocator
tags: allocation, arena, bump-allocator, alignment
track: hft
difficulty: medium

A monotonic allocator over one fixed byte buffer: each request bumps a cursor forward and returns an aligned offset; there is no per-object free, only a bulk `reset()` that rewinds the cursor to zero. Implement `size_t allocate(size_t size, size_t align)` returning the aligned offset of a `size`-byte block (or `FAIL` if it would overflow the buffer), `void reset()`, and `size_t used() const`. Allocation is a couple of integer ops — round the cursor up to the alignment, hand back that offset, advance past the block.

Constraints: `Capacity` is a compile-time constant. `align` is a power of two. No heap allocation — the arena owns a fixed byte array. Returned offsets must satisfy `offset % align == 0` and blocks must not overlap.

Example: on `BumpArena<64>`, `allocate(1,1)` returns `0` (cursor -> 1); `allocate(8,8)` rounds up and returns `8` (cursor -> 16); `allocate(4,4)` returns `16` (cursor -> 20); `allocate(64,1)` overflows and returns `FAIL`; `used()==20`; after `reset()`, `allocate(1,1)` returns `0` again.

hint: Rounding an offset up to a power-of-two alignment is `(offset + align - 1) & ~(align - 1)` — no division needed.
hint: Compute the aligned start first, then check `aligned + size <= Capacity` before committing; only advance the cursor on success.
hint: `reset()` is the whole deallocation story — set the cursor back to 0 and the entire buffer is free again in O(1).

```cpp
// starter
template <size_t Capacity>
struct BumpArena {
    alignas(std::max_align_t) unsigned char buf_[Capacity];
    size_t offset_ = 0;
    static constexpr size_t FAIL = ~size_t(0);
    // implement allocate / reset / used
};
```

```cpp
size_t allocate(size_t size, size_t align) {
    size_t aligned = (offset_ + (align - 1)) & ~(align - 1);
    if (aligned + size > Capacity) return FAIL;
    offset_ = aligned + size;
    return aligned;
}
void reset() { offset_ = 0; }
size_t used() const { return offset_; }
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <size_t Capacity>
struct BumpArena {
    alignas(std::max_align_t) unsigned char buf_[Capacity];
    size_t offset_ = 0;
    static constexpr size_t FAIL = ~size_t(0);
    //__USER__
};
int main() {
    BumpArena<64> a;
    if (a.used() != 0) { std::puts("init used"); return 1; }
    size_t o1 = a.allocate(1, 1);
    if (o1 != 0 || a.used() != 1) { std::puts("first alloc"); return 1; }
    size_t o2 = a.allocate(8, 8);              // must round 1 up to 8
    if (o2 != 8 || (o2 % 8) != 0 || a.used() != 16) { std::puts("aligned alloc"); return 1; }
    size_t o3 = a.allocate(4, 4);
    if (o3 != 16 || (o3 % 4) != 0 || a.used() != 20) { std::puts("third alloc"); return 1; }
    // blocks must not overlap: o2's [8,16) sits after o1's [0,1), o3 starts at 16
    if (!(o1 + 1 <= o2 && o2 + 8 <= o3)) { std::puts("blocks overlap"); return 1; }
    if (a.allocate(64, 1) != BumpArena<64>::FAIL) { std::puts("overflow must return FAIL"); return 1; }
    if (a.used() != 20) { std::puts("failed alloc must not advance cursor"); return 1; }
    size_t o4 = a.allocate(16, 16);            // round 20 up to 32
    if (o4 != 32 || (o4 % 16) != 0) { std::puts("16-byte alignment"); return 1; }
    a.reset();
    if (a.used() != 0) { std::puts("reset must rewind"); return 1; }
    if (a.allocate(1, 1) != 0) { std::puts("reuse after reset"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A bump allocator is the fastest possible allocator: `allocate` is an align-and-add on a single cursor, and freeing is deferred to a bulk `reset()` that costs one store. That is exactly the shape you want for per-tick or per-message scratch memory — carve out everything you need for one unit of work from a pre-sized arena, then rewind at the top of the next iteration. Compared to `malloc`/`new`, there is no free-list search, no header per allocation, no fragmentation, and no lock; the arena is one contiguous, cache-friendly block. The one thing you must get right is alignment: rounding up with `(off + align-1) & ~(align-1)` keeps every returned offset suitably aligned for the type you'll placement-construct there, and checking `aligned + size <= Capacity` before committing keeps the cursor honest on overflow.
