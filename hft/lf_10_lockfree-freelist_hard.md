## challenge: Fixed-size lock-free freelist
tags: lock-free, freelist, cas
track: hft
difficulty: hard

A lock-free **index allocator**: hand out and reclaim slots of a fixed pool of `N` objects (message buffers, order records) with no `malloc` on the hot path. Instead of linking free *pointers* (which invites ABA and reclamation hazards), thread a stack through an `int next_[N]` array and keep an `std::atomic<int> head_` holding the index of the first free slot (`-1` = exhausted). Initialize the chain `0→1→…→N-1→-1` with `head_ = 0`. `alloc()` CAS-pops the head, returning its index or `-1`; `free(i)` CAS-pushes `i` back on. Implement the constructor plus `int alloc()` and `void free(int i)`. Single-threaded correctness; the harness checks exhaustion and reuse.

Constraints: no dynamic allocation, no locks. `alloc` returns `-1` when the pool is empty; freed indices become available again.

Example: with `N = 8`, eight `alloc()` calls yield eight distinct indices and the ninth returns `-1`; freeing two indices lets exactly two more `alloc()` calls succeed, reusing those slots.

hint: `alloc` is a Treiber pop over indices: load `h = head_`; if `-1` return it; else CAS `head_` from `h` to `next_[h]`, looping on failure.
hint: `free(i)` is a Treiber push: set `next_[i] = head_`, then CAS `head_` to `i`; retry if the head moved under you.
hint: `alloc` should `acquire` (the slot's contents were published by whoever `free`d it) and `free` should `release`; the failing CAS legs need only `relaxed`.

```cpp
// starter
#include <atomic>
#include <cstddef>
using std::size_t;
template <size_t N>
struct FreeList {
    std::atomic<int> head_;
    int next_[N];
    FreeList();          // chain 0->1->...->N-1->-1, head_ = 0
    int alloc();         // returns a free index, or -1 if exhausted
    void free(int i);    // returns index i to the pool
};
```

```cpp
FreeList() {
    for (size_t i = 0; i + 1 < N; ++i) next_[i] = (int)(i + 1);
    next_[N - 1] = -1;
    head_.store(0, std::memory_order_relaxed);
}
int alloc() {
    int h = head_.load(std::memory_order_acquire);
    while (h != -1 && !head_.compare_exchange_weak(h, next_[h],
               std::memory_order_acquire, std::memory_order_relaxed)) {
        // CAS refreshed h to the current head; retry
    }
    return h;
}
void free(int i) {
    int h = head_.load(std::memory_order_relaxed);
    do {
        next_[i] = h;
    } while (!head_.compare_exchange_weak(h, i,
               std::memory_order_release, std::memory_order_relaxed));
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <cstddef>
using std::size_t;
template <size_t N>
struct FreeList {
    std::atomic<int> head_;
    int next_[N];
    //__USER__
};
int main() {
    FreeList<8> fl;
    bool seen[8] = {false};
    int idx[8];
    for (int i = 0; i < 8; ++i) {
        idx[i] = fl.alloc();
        if (idx[i] < 0 || idx[i] >= 8 || seen[idx[i]]) { std::puts("alloc must yield distinct valid indices"); return 1; }
        seen[idx[i]] = true;
    }
    if (fl.alloc() != -1) { std::puts("exhausted must return -1"); return 1; }
    fl.free(idx[3]);
    fl.free(idx[5]);
    int a = fl.alloc();
    int b = fl.alloc();
    if (fl.alloc() != -1) { std::puts("exhausted again must return -1"); return 1; }
    bool ok = (a == idx[5] && b == idx[3]) || (a == idx[3] && b == idx[5]);
    if (!ok) { std::puts("must reuse freed indices"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A freelist is a Treiber stack in disguise, and the interesting design decision is storing *indices* rather than pointers. The `next` links live in a side array `next_[]` indexed by slot number, and `head_` is a plain `int`, so `alloc`/`free` are the same load-check-CAS loops as the pointer stack but operate on small integers. Two payoffs: allocation is O(1) with zero `malloc`, and — because an `int` index is not a reusable heap address — you dodge the worst of the pointer stack's **ABA problem** and its reclamation hazard (there is no node to free; the storage is the pool itself, which outlives every operation). ABA can still bite in principle if a slot is popped, pushed, and re-popped between one thread's load and CAS, so production allocators fold a version tag into a double-width `head_` (a packed `{index, tag}` swapped with a 64-bit or `cmpxchg16b` CAS); this single-threaded version doesn't need it. Minimal-correct ordering: `alloc`'s successful CAS is `acquire` so a reused slot's contents (written by the previous owner before it called `free`) are visible, and `free`'s CAS is `release` to publish them — a release/acquire pair through `head_`. The failing legs are `relaxed`. `alloc` returning `-1` on an empty list is the graceful "pool exhausted" signal a fixed-size design must always provide.
