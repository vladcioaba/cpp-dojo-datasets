## challenge: Fixed-capacity stack
tags: allocation, stack, lifo, cache-locality
track: hft
difficulty: easy

A LIFO stack with a compile-time capacity and no dynamic memory. Store elements in one array and keep a single size cursor. Implement `bool push(const T&)` (returns `false` when full), `bool pop(T& out)` (returns `false` when empty, else writes the popped top), `bool top(T& out) const` (peek without removing), plus `full()`, `empty()`, and `size()`. Every operation is O(1) and touches only the top of the array.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. No heap allocation, no `std::` containers.

Example: on `FixedStack<int,2>`, `push(7); push(8)` fills it; a 3rd `push` returns `false`; `top` reads `8`; `pop` yields `8` then `7` (LIFO).

hint: A stack needs exactly one piece of mutable state beyond the storage: how many elements are live.
hint: The top element is always at index `size_ - 1`; push writes there after incrementing, pop reads there before decrementing.
hint: `push` writing `buf_[size_++]` and `pop` reading `buf_[--size_]` are mirror images — no element is ever moved.

```cpp
// starter
template <class T, size_t N>
struct FixedStack {
    T buf_[N];
    size_t size_ = 0;
    // implement push / pop / top / full / empty / size
};
```

```cpp
bool empty() const { return size_ == 0; }
bool full()  const { return size_ == N; }
size_t size() const { return size_; }
bool push(const T& v) {
    if (size_ == N) return false;
    buf_[size_++] = v;
    return true;
}
bool pop(T& out) {
    if (size_ == 0) return false;
    out = buf_[--size_];
    return true;
}
bool top(T& out) const {
    if (size_ == 0) return false;
    out = buf_[size_ - 1];
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct FixedStack {
    T buf_[N];
    size_t size_ = 0;
    //__USER__
};
int main() {
    FixedStack<int, 2> s;
    if (!s.empty() || s.full() || s.size() != 0) { std::puts("init state"); return 1; }
    int x;
    if (s.pop(x)) { std::puts("pop on empty must return false"); return 1; }
    if (s.top(x)) { std::puts("top on empty must return false"); return 1; }
    if (!s.push(7) || !s.push(8)) { std::puts("push to full failed"); return 1; }
    if (!s.full() || s.size() != 2) { std::puts("should be full"); return 1; }
    if (s.push(9)) { std::puts("push on full must return false"); return 1; }
    if (!s.top(x) || x != 8) { std::puts("top should read 8"); return 1; }
    if (s.size() != 2) { std::puts("top must not pop"); return 1; }
    if (!s.pop(x) || x != 8) { std::puts("LIFO order at 8"); return 1; }
    if (!s.pop(x) || x != 7) { std::puts("LIFO order at 7"); return 1; }
    if (!s.empty()) { std::puts("should be empty"); return 1; }
    if (s.pop(x)) { std::puts("pop on empty again"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A stack is the simplest allocation-free container: one array plus a size cursor. Because pushes and pops only ever touch `buf_[size_]`, the working set stays hot in L1 and the branch predictor sees a trivial full/empty test. This is what you reach for instead of `std::stack<std::deque>` on the hot path — no per-element node, no allocator call, no pointer indirection. The whole structure is `N * sizeof(T) + sizeof(size_t)` bytes, entirely on the stack or inside a parent object, with O(1) operations and zero heap traffic.
