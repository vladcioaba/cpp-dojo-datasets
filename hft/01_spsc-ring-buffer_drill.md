## challenge: SPSC ring buffer
tags: lock-free, spsc, ring-buffer
track: hft

The bread-and-butter HFT structure: a single-producer, single-consumer queue with no locks. Buffer size `N` is a power of two. Track fill with **monotonically increasing** `head_` (pop index) and `tail_` (push index) of type `size_t`; the usable slot is `buf_[i & (N - 1)]`. Implement `bool push(const T& v)` (fails, returns false, when full — size `N`) and `bool pop(T& out)` (fails when empty). Single-threaded correctness only here; the real thing pairs `release`/`acquire` on the indices.

hint: Because `head_` and `tail_` only ever increase, their difference `tail_ - head_` is exactly the number of unread elements.
hint: Since N is a power of two, `i & (N - 1)` is the cheap replacement for `i % N` when mapping an index to a slot.
hint: Full means the difference equals N; empty means `head_ == tail_`. Never reset the counters — let unsigned arithmetic wrap them naturally.

```cpp
// starter
template <class T, size_t N>
struct SpscQueue {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    T      buf_[N];
    size_t head_ = 0;   // next to pop
    size_t tail_ = 0;   // next to push
    // implement push / pop
};
```

```cpp
bool push(const T& v) {
    if (tail_ - head_ == N) return false;   // full
    buf_[tail_ & (N - 1)] = v;
    ++tail_;
    return true;
}
bool pop(T& out) {
    if (tail_ == head_) return false;        // empty
    out = buf_[head_ & (N - 1)];
    ++head_;
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdlib>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct SpscQueue {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    T      buf_[N];
    size_t head_ = 0;
    size_t tail_ = 0;
    //__USER__
};
int main() {
    SpscQueue<int, 4> q;
    int x = 0;
    if (q.pop(x)) { std::puts("pop on empty must fail"); return 1; }
    for (int i = 0; i < 4; ++i) if (!q.push(i)) { std::puts("push should fit 4"); return 1; }
    if (q.push(99)) { std::puts("push on full must fail"); return 1; }
    for (int i = 0; i < 4; ++i) {
        if (!q.pop(x) || x != i) { std::puts("FIFO order broken"); return 1; }
    }
    if (q.pop(x)) { std::puts("empty again must fail"); return 1; }
    // wrap-around past the buffer end
    for (int r = 0; r < 10; ++r) {
        if (!q.push(r) || !q.pop(x) || x != r) { std::puts("wrap-around broken"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Monotonic `head_`/`tail_` counters turn full/empty checks into a subtraction: `tail_ - head_` is the current size, so full is `== N` and empty is `head_ == tail_`. Masking with `N - 1` (valid because N is a power of two) indexes the slot, and unsigned wraparound keeps this correct forever. Push and pop are O(1); storage is O(N). The concurrent version pairs release/acquire on the indices to publish writes safely.
