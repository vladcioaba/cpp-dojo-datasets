## challenge: Fixed-capacity ring buffer
tags: allocation, ring-buffer, circular-buffer, cache-locality
track: hft
difficulty: easy

A single-threaded FIFO queue that never allocates. Back it with one contiguous array of `N` slots and two cursors — the index of the front element and a live count. Implement `bool push(const T&)` (returns `false` when full), `bool pop(T& out)` (returns `false` when empty and otherwise writes the front element), plus `full()`, `empty()`, and `size()`. Wrapping is done with modular arithmetic so the buffer reuses the same memory indefinitely.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. No heap allocation, no `std::` containers. All operations are O(1).

Example: on `RingBuffer<int,3>`, `push(1); push(2); push(3)` fills it; a 4th `push` returns `false`; `pop` yields `1` then `2` (FIFO); after popping you can `push` again and the storage wraps around.

hint: Track the front index and a count instead of a front and a back pointer — it removes the "is it full or empty?" ambiguity when the two cursors coincide.
hint: The physical slot of the k-th element is `(head_ + k) % N`; the tail you write to is `(head_ + count_) % N`.
hint: `push` bumps the count, `pop` advances `head_` modulo `N` and drops the count — neither ever moves existing elements.

```cpp
// starter
template <class T, size_t N>
struct RingBuffer {
    T buf_[N];
    size_t head_ = 0;    // index of the front element
    size_t count_ = 0;   // number of live elements
    // implement push / pop / full / empty / size
};
```

```cpp
bool empty() const { return count_ == 0; }
bool full()  const { return count_ == N; }
size_t size() const { return count_; }
bool push(const T& v) {
    if (count_ == N) return false;
    buf_[(head_ + count_) % N] = v;
    ++count_;
    return true;
}
bool pop(T& out) {
    if (count_ == 0) return false;
    out = buf_[head_];
    head_ = (head_ + 1) % N;
    --count_;
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct RingBuffer {
    T buf_[N];
    size_t head_ = 0;
    size_t count_ = 0;
    //__USER__
};
int main() {
    RingBuffer<int, 3> r;
    if (!r.empty() || r.full() || r.size() != 0) { std::puts("init state"); return 1; }
    if (!r.push(1) || !r.push(2) || !r.push(3)) { std::puts("push to full failed"); return 1; }
    if (!r.full() || r.size() != 3) { std::puts("should be full"); return 1; }
    if (r.push(4)) { std::puts("push on full must return false"); return 1; }
    int x;
    if (!r.pop(x) || x != 1) { std::puts("FIFO order at 1"); return 1; }
    if (!r.pop(x) || x != 2) { std::puts("FIFO order at 2"); return 1; }
    // wrap: writing past the physical end reuses freed slots
    if (!r.push(5) || !r.push(6)) { std::puts("wrap push failed"); return 1; }
    if (r.size() != 3) { std::puts("size after wrap"); return 1; }
    if (!r.pop(x) || x != 3) { std::puts("FIFO order at 3"); return 1; }
    if (!r.pop(x) || x != 5) { std::puts("wrap order at 5"); return 1; }
    if (!r.pop(x) || x != 6) { std::puts("wrap order at 6"); return 1; }
    if (!r.empty()) { std::puts("should be empty"); return 1; }
    if (r.pop(x)) { std::puts("pop on empty must return false"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A `head + count` circular buffer is the canonical allocation-free FIFO. The elements live in one cache-friendly array, so pushes and pops touch adjacent memory and never chase pointers; contrast `std::queue<std::deque>`, whose block-of-blocks layout costs an allocation per growth and scatters elements across the heap. Using a live `count_` rather than a separate tail cursor removes the classic full-vs-empty ambiguity when the two indices meet, keeping every operation branch-light and O(1) with zero heap traffic on the hot path.
