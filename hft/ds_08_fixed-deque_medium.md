## challenge: Fixed-capacity double-ended queue
tags: allocation, deque, ring-buffer, cache-locality
track: hft
difficulty: medium

A double-ended queue with a compile-time capacity, backed by a single circular array — push and pop at both ends in O(1) with no allocation. Implement `bool push_front(const T&)` and `bool push_back(const T&)` (each returns `false` when full), `bool pop_front(T& out)` and `bool pop_back(T& out)` (each returns `false` when empty, else writes the removed element), plus `full()`, `empty()`, and `size()`. Track a front index and a live count; both ends wrap with modular arithmetic.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. No heap allocation. All four push/pop operations are O(1) and move no existing elements.

Example: on `Deque<int,3>`, `push_back(1); push_back(2); push_front(0)` fills it as `[0,1,2]`; further pushes at either end return `false`; `pop_front` yields `0`, `pop_back` yields `2`, `pop_front` yields `1`.

hint: A front index plus a count describes the whole deque; the back element is at `(head_ + count_ - 1) % N`.
hint: `push_front` moves the head one slot backward — `(head_ + N - 1) % N` — then writes there; `push_back` writes at the tail `(head_ + count_) % N`.
hint: `pop_front` advances the head forward modulo `N`; `pop_back` just drops the count. Neither shifts elements.

```cpp
// starter
template <class T, size_t N>
struct Deque {
    T buf_[N];
    size_t head_ = 0;    // index of the front element
    size_t count_ = 0;   // number of live elements
    // implement push_front / push_back / pop_front / pop_back / full / empty / size
};
```

```cpp
bool empty() const { return count_ == 0; }
bool full()  const { return count_ == N; }
size_t size() const { return count_; }
bool push_back(const T& v) {
    if (count_ == N) return false;
    buf_[(head_ + count_) % N] = v;
    ++count_;
    return true;
}
bool push_front(const T& v) {
    if (count_ == N) return false;
    head_ = (head_ + N - 1) % N;
    buf_[head_] = v;
    ++count_;
    return true;
}
bool pop_front(T& out) {
    if (count_ == 0) return false;
    out = buf_[head_];
    head_ = (head_ + 1) % N;
    --count_;
    return true;
}
bool pop_back(T& out) {
    if (count_ == 0) return false;
    out = buf_[(head_ + count_ - 1) % N];
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
struct Deque {
    T buf_[N];
    size_t head_ = 0;
    size_t count_ = 0;
    //__USER__
};
int main() {
    Deque<int, 3> d;
    if (!d.empty() || d.full() || d.size() != 0) { std::puts("init state"); return 1; }
    int x;
    if (d.pop_front(x) || d.pop_back(x)) { std::puts("pop on empty must fail"); return 1; }
    if (!d.push_back(1) || !d.push_back(2)) { std::puts("push_back failed"); return 1; }
    if (!d.push_front(0)) { std::puts("push_front failed"); return 1; }   // [0,1,2]
    if (!d.full() || d.size() != 3) { std::puts("should be full"); return 1; }
    if (d.push_back(9) || d.push_front(9)) { std::puts("push on full must fail"); return 1; }
    if (!d.pop_front(x) || x != 0) { std::puts("pop_front should give 0"); return 1; }
    if (!d.pop_back(x)  || x != 2) { std::puts("pop_back should give 2"); return 1; }
    if (!d.pop_front(x) || x != 1) { std::puts("pop_front should give 1"); return 1; }
    if (!d.empty()) { std::puts("should be empty"); return 1; }
    // exercise head wraparound via repeated push_front
    if (!d.push_front(5) || !d.push_front(6) || !d.push_front(7)) { std::puts("wrap push_front"); return 1; }
    if (d.size() != 3) { std::puts("size after wrap"); return 1; }         // front->back: 7,6,5
    if (!d.pop_front(x) || x != 7) { std::puts("wrap pop_front 7"); return 1; }
    if (!d.pop_back(x)  || x != 5) { std::puts("wrap pop_back 5"); return 1; }
    if (!d.pop_front(x) || x != 6) { std::puts("wrap pop_front 6"); return 1; }
    if (!d.empty()) { std::puts("empty after wrap drain"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A fixed deque is a ring buffer that grows and shrinks at both ends. The trick is the same `head + count` bookkeeping as a plain circular queue, extended with a backward step for `push_front` (`(head - 1) mod N`, written as `+ N - 1` to avoid unsigned underflow). Every operation is a modular index update over one contiguous array, so no element ever moves and there is no allocation — unlike `std::deque`, whose segmented block-of-blocks layout allocates blocks on demand, scatters them across the heap, and pays a double indirection on access. When you need bounded front-and-back access on the hot path (a bounded work queue, a sliding window you trim from either side), this keeps the whole structure in a few cache lines with O(1) branch-light operations.
