## challenge: Fixed-capacity object pool
tags: allocation, object-pool, memory
track: hft

No `new` on the hot path. Pre-allocate `N` slots, hand them out from a free list. Implement `T* alloc()` (returns `nullptr` when exhausted) and `void release(T* p)` (returns a slot to the pool). Use an intrusive free-list index stack over the storage — no allocation, O(1) both ways.

```cpp
// starter
template <class T, size_t N>
struct Pool {
    alignas(T) unsigned char storage_[N * sizeof(T)];
    size_t free_[N];
    size_t top_ = N;                 // free_[0..top_) hold free indices
    Pool() { for (size_t i = 0; i < N; ++i) free_[i] = N - 1 - i; }
    T* slot(size_t i) { return reinterpret_cast<T*>(storage_) + i; }
    // implement alloc / release
};
```

```cpp
T* alloc() {
    if (top_ == 0) return nullptr;
    return slot(free_[--top_]);
}
void release(T* p) {
    size_t i = static_cast<size_t>(p - slot(0));
    free_[top_++] = i;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <class T, size_t N>
struct Pool {
    alignas(T) unsigned char storage_[N * sizeof(T)];
    size_t free_[N];
    size_t top_ = N;
    Pool() { for (size_t i = 0; i < N; ++i) free_[i] = N - 1 - i; }
    T* slot(size_t i) { return reinterpret_cast<T*>(storage_) + i; }
    //__USER__
};
int main() {
    Pool<long, 3> p;
    long* a = p.alloc();
    long* b = p.alloc();
    long* c = p.alloc();
    if (!a || !b || !c) { std::puts("first 3 allocs must succeed"); return 1; }
    if (p.alloc() != nullptr) { std::puts("exhausted pool must return nullptr"); return 1; }
    // pointers must land inside storage and be distinct
    if (a == b || b == c || a == c) { std::puts("slots must be distinct"); return 1; }
    p.release(b);
    long* d = p.alloc();
    if (d != b) { std::puts("released slot should be reused"); return 1; }
    if (p.alloc() != nullptr) { std::puts("full again must return nullptr"); return 1; }
    std::puts("PASS");
}
```
