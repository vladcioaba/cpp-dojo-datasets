## challenge: Power-of-two ring buffer
tags: ring-buffer, power-of-two, spsc
track: hft
difficulty: medium

Ring buffers are everywhere in low-latency code (event queues, market-data pipes). Give one a power-of-two capacity and index wrap-around becomes a single bitwise `&` instead of a `%` — no division on the hot path. Implement `RingBuffer<T, N>` with inline storage for `N` elements (`N` a power of two, enforced at compile time): `push(v)` appends unless full (returns `false`), `pop(out)` removes the oldest unless empty (returns `false`), `size()` reports the count. Use monotonically increasing head/tail counters masked by `N - 1` to index the storage.

Constraints: `N` must be a power of two — reject anything else with a `static_assert`. No dynamic allocation; storage is a fixed `T[N]` member. FIFO order.

Example: with `RingBuffer<int, 4>`, four pushes succeed and a fifth fails; four pops return the values in the order pushed; pushing and popping repeatedly wraps the indices past `N` while staying correct.

hint: Keep two ever-increasing counters, `head_` (next to pop) and `tail_` (next to push); the live count is `tail_ - head_`, full is `count == N`, empty is `head_ == tail_`.
hint: Because `N` is a power of two, the physical slot is `index & (N - 1)` — the mask replaces `index % N` with one AND instruction.
hint: Guard `N` with `static_assert((N & (N - 1)) == 0, ...)` so a non-power-of-two capacity fails to compile.

```cpp
// starter
#include <cstddef>
template <typename T, std::size_t N>
class RingBuffer {
public:
    bool push(const T& v);   // false if full
    bool pop(T& out);        // false if empty
    std::size_t size() const;
};
```

```cpp
template <typename T, std::size_t N>
class RingBuffer {
    static_assert(N >= 2 && (N & (N - 1)) == 0, "capacity must be a power of two");
    alignas(64) T buf_[N];
    std::size_t head_ = 0;   // next index to pop
    std::size_t tail_ = 0;   // next index to push
public:
    bool push(const T& v) {
        if (tail_ - head_ == N) return false;    // full
        buf_[tail_ & (N - 1)] = v;
        ++tail_;
        return true;
    }
    bool pop(T& out) {
        if (head_ == tail_) return false;         // empty
        out = buf_[head_ & (N - 1)];
        ++head_;
        return true;
    }
    std::size_t size() const { return tail_ - head_; }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
//__USER__
int main() {
    RingBuffer<int, 4> rb;
    if (rb.size() != 0) { std::puts("should start empty"); return 1; }

    int tmp = -1;
    if (rb.pop(tmp)) { std::puts("pop on empty should fail"); return 1; }

    for (int i = 0; i < 4; ++i)
        if (!rb.push(i * 10)) { std::puts("push within capacity should succeed"); return 1; }
    if (rb.size() != 4) { std::puts("size should be 4"); return 1; }
    if (rb.push(999)) { std::puts("push on full should fail"); return 1; }

    // FIFO order.
    for (int i = 0; i < 4; ++i) {
        if (!rb.pop(tmp) || tmp != i * 10) { std::puts("wrong FIFO order"); return 1; }
    }
    if (rb.size() != 0) { std::puts("should be empty again"); return 1; }

    // Wrap-around: counters climb past N, masking must keep indexing correct.
    for (int round = 0; round < 3; ++round) {
        for (int i = 0; i < 4; ++i) rb.push(round * 100 + i);
        for (int i = 0; i < 4; ++i) {
            rb.pop(tmp);
            if (tmp != round * 100 + i) { std::puts("wrap-around broke ordering"); return 1; }
        }
    }
    std::puts("PASS");
}
```

**Editorial:** Storing `head_`/`tail_` as unbounded increasing counters makes the occupancy math trivial: `tail_ - head_` is the live count, so full and empty are distinguishable without wasting a slot or keeping a separate size flag. The power-of-two capacity is the key optimization — the physical slot is `index & (N - 1)`, a single AND, versus a costly `%` for arbitrary `N`. The `static_assert` makes a bad capacity a compile error rather than a silent correctness bug. `alignas(64)` on the storage keeps the buffer off shared cache lines. This exact structure underlies lock-free SPSC queues on the market-data hot path.
