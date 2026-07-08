## challenge: SPSC ring with masked head/tail
tags: lock-free, spsc, ring-buffer
track: hft
difficulty: medium

A variant of the SPSC queue that keeps `head_`/`tail_` as **already-masked positions** in `[0, N)` rather than free-running counters. Buffer size `N` is a power of two; `tail_` is the write slot, `head_` the read slot, and both advance with `(i + 1) & (N - 1)`. To tell *full* from *empty* when both indices can be equal, sacrifice one slot: the queue is **empty** when `head_ == tail_` and **full** when advancing `tail_` would collide with `head_` — so usable capacity is `N - 1`. Implement `bool push(const T&)` and `bool pop(T&)`. Single-threaded correctness; the concurrent version pairs `release`/`acquire` on the two indices.

Constraints: `N` a power of two, indices stored pre-masked in `[0, N)`. Usable capacity is `N - 1`, not `N`.

Example: with `N = 4`, three pushes succeed and the fourth fails (one slot reserved); pop returns them FIFO; the ring wraps cleanly across the buffer end forever.

hint: The next write position is `next = (tail_ + 1) & (N - 1)`; the ring is full precisely when `next == head_`.
hint: Empty is simply `head_ == tail_` — the reserved slot guarantees full and empty never look the same.
hint: The producer publishes data by `release`-storing the new `tail_`; the consumer `acquire`-loads `tail_` so it never reads a slot before the value landed (and symmetrically for `head_`).

```cpp
// starter
#include <atomic>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct SpscRing {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    T buf_[N];
    std::atomic<size_t> head_{0};   // read position, pre-masked
    std::atomic<size_t> tail_{0};   // write position, pre-masked
    bool push(const T& v);
    bool pop(T& out);
};
```

```cpp
bool push(const T& v) {
    size_t t = tail_.load(std::memory_order_relaxed);
    size_t next = (t + 1) & (N - 1);
    if (next == head_.load(std::memory_order_acquire)) return false;   // full
    buf_[t] = v;
    tail_.store(next, std::memory_order_release);
    return true;
}
bool pop(T& out) {
    size_t h = head_.load(std::memory_order_relaxed);
    if (h == tail_.load(std::memory_order_acquire)) return false;      // empty
    out = buf_[h];
    head_.store((h + 1) & (N - 1), std::memory_order_release);
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct SpscRing {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    T buf_[N];
    std::atomic<size_t> head_{0};
    std::atomic<size_t> tail_{0};
    //__USER__
};
int main() {
    SpscRing<int, 4> q;   // usable capacity N-1 == 3
    int x = 0;
    if (q.pop(x)) { std::puts("pop on empty must fail"); return 1; }
    for (int i = 0; i < 3; ++i) if (!q.push(i)) { std::puts("must fit 3"); return 1; }
    if (q.push(99)) { std::puts("full at N-1 must fail"); return 1; }
    for (int i = 0; i < 3; ++i) { if (!q.pop(x) || x != i) { std::puts("FIFO order broken"); return 1; } }
    if (q.pop(x)) { std::puts("empty again must fail"); return 1; }
    for (int r = 0; r < 20; ++r) { if (!q.push(r) || !q.pop(x) || x != r) { std::puts("wrap-around broken"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is the "masked-index" ring, contrasted with the monotonic-counter ring where `head_`/`tail_` grow without bound and you compute size as `tail_ - head_`. Storing pre-masked positions keeps the indices small but reintroduces the classic ambiguity: `head_ == tail_` could mean either empty or completely full. The standard fix is to keep one slot always empty, so full is detected *before* the collision — `(tail_ + 1) & (N - 1) == head_` — at the cost of one unusable slot (capacity `N - 1`). That is the trade against the counter version, which uses all `N` slots because subtraction distinguishes the two states. Masking with `N - 1` is the cheap `% N` and is why `N` must be a power of two. Minimal-correct ordering for the real SPSC case: the producer `release`-stores `tail_` after writing the slot, and the consumer `acquire`-loads `tail_` before reading it, so the data write happens-before the read; `head_` carries the symmetric handshake so the producer never overwrites an unread slot. Each side only writes its own index, so no CAS is needed — that single-writer-per-variable property is what makes SPSC the fastest lock-free queue.
