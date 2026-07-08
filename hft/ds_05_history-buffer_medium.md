## challenge: Fixed history buffer (last-N values)
tags: allocation, ring-buffer, sliding-window, cache-locality
track: hft
difficulty: medium

A rolling window that always remembers the most recent `N` values pushed and silently forgets older ones. Back it with a circular array and a write cursor. Implement `push(const T&)` (overwrites the oldest slot once full), `size()` (number stored, capped at `N`), and `recent(i)` — the i-th most recent value, where `recent(0)` is the newest and `recent(size()-1)` is the oldest still retained. No allocation and no shifting: pushing is O(1) even though the logical window slides on every call.

Constraints: `N` is a compile-time constant, `1 <= N`. For `recent(i)`, `0 <= i < size()`. No heap allocation. Order of retrieval must reflect recency, not physical slot order.

Example: on `History<int,3>`, after `push(1); push(2); push(3); push(4)` the window holds the last 3: `size()==3`, `recent(0)==4`, `recent(1)==3`, `recent(2)==2` (the `1` was overwritten).

hint: Keep a cursor to where the next value will be written; the newest value is one slot behind it, modulo `N`.
hint: Never shift elements — pushing just writes at the cursor, advances it modulo `N`, and grows the count until it saturates at `N`.
hint: `recent(i)` maps recency to a physical slot: start from the newest index `(next_ + N - 1)` and walk back by `i`, all modulo `N` (add `N` before the modulo to stay non-negative).

```cpp
// starter
template <class T, size_t N>
struct History {
    T buf_[N];
    size_t next_ = 0;    // slot the next push writes to
    size_t count_ = 0;   // values retained, saturates at N
    // implement push / size / recent
};
```

```cpp
void push(const T& v) {
    buf_[next_] = v;
    next_ = (next_ + 1) % N;
    if (count_ < N) ++count_;
}
size_t size() const { return count_; }
T recent(size_t i) const {
    // i == 0 -> newest (slot next_-1); larger i -> older
    return buf_[(next_ + N - 1 - i) % N];
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct History {
    T buf_[N];
    size_t next_ = 0;
    size_t count_ = 0;
    //__USER__
};
int main() {
    History<int, 3> h;
    if (h.size() != 0) { std::puts("init size"); return 1; }
    h.push(1); h.push(2);
    if (h.size() != 2) { std::puts("size before full"); return 1; }
    if (h.recent(0) != 2 || h.recent(1) != 1) { std::puts("recency before full"); return 1; }
    h.push(3);
    if (h.size() != 3) { std::puts("size at capacity"); return 1; }
    if (h.recent(0) != 3 || h.recent(1) != 2 || h.recent(2) != 1) { std::puts("recency at capacity"); return 1; }
    h.push(4);                                  // overwrites oldest (1)
    if (h.size() != 3) { std::puts("size stays capped"); return 1; }
    if (h.recent(0) != 4 || h.recent(1) != 3 || h.recent(2) != 2) { std::puts("recency after overwrite"); return 1; }
    h.push(5); h.push(6);                        // more wraps
    if (h.recent(0) != 6 || h.recent(1) != 5 || h.recent(2) != 4) { std::puts("recency after wraps"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A last-N window is a circular buffer read newest-first. Because pushing overwrites the slot the cursor points at and just advances the cursor modulo `N`, every update is O(1) with no element movement — unlike a naive "erase front, append back" on a `std::deque` or a shifting array, which is O(N) per tick and churns the allocator. The fixed array is one contiguous, cache-resident block, so scanning the recent window (moving averages, last-N trades, a rolling checksum) streams through memory. The only subtlety is the index math in `recent()`: recency `i` maps to physical slot `(next_ - 1 - i) mod N`, computed with a `+N` bias so the subtraction never underflows the unsigned index.
