## challenge: Inline small_vector<T,N>
tags: allocation, small-vector, inline-storage, cache-locality
track: hft
difficulty: medium

A growable-looking vector whose entire capacity lives inline — no heap, ever. Storage is one array of `N` elements plus a size. Implement `bool push_back(const T&)` (returns `false` at capacity instead of allocating), `size()`, `full()`, mutable and const `operator[]`, `back()`, and `clear()`. The point is a `std::vector`-like API with `std::array`-like locality: the object owns its bytes, so copying or embedding it moves the data with it and never touches the allocator.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. No heap allocation, no `std::vector`. Indexing is unchecked (caller guarantees `i < size()`), matching `operator[]`'s contract.

Example: on `SmallVector<int,4>`, `push_back(10); push_back(20)` gives `size()==2`, `v[0]==10`, `v[1]==20`; `v[0] = 99` mutates in place; a 5th `push_back` on a full vector returns `false`; `clear()` resets `size()` to 0 without releasing memory.

hint: The only difference from a stack is the random-access interface — but the storage discipline (array + size, append at the end) is identical.
hint: `operator[]` returns a reference into `buf_`; provide both a non-const overload (for assignment) and a const overload (for read-only access).
hint: `clear()` need not touch the elements for trivial `T` — resetting `size_` to 0 logically empties the vector with no work.

```cpp
// starter
template <class T, size_t N>
struct SmallVector {
    T buf_[N];
    size_t size_ = 0;
    // implement push_back / size / full / operator[] (x2) / back / clear
};
```

```cpp
size_t size() const { return size_; }
bool full() const { return size_ == N; }
bool push_back(const T& v) {
    if (size_ == N) return false;
    buf_[size_++] = v;
    return true;
}
T& operator[](size_t i) { return buf_[i]; }
const T& operator[](size_t i) const { return buf_[i]; }
T& back() { return buf_[size_ - 1]; }
void clear() { size_ = 0; }
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <class T, size_t N>
struct SmallVector {
    T buf_[N];
    size_t size_ = 0;
    //__USER__
};
static int sum_const(const SmallVector<int, 4>& v) {
    int s = 0;
    for (size_t i = 0; i < v.size(); ++i) s += v[i];   // exercises const operator[]
    return s;
}
int main() {
    SmallVector<int, 4> v;
    if (v.size() != 0 || v.full()) { std::puts("init state"); return 1; }
    if (!v.push_back(10) || !v.push_back(20) || !v.push_back(30)) { std::puts("push_back failed"); return 1; }
    if (v.size() != 3) { std::puts("size wrong"); return 1; }
    if (v[0] != 10 || v[1] != 20 || v[2] != 30) { std::puts("read via operator[]"); return 1; }
    v[0] = 99;                                // exercises mutable operator[]
    if (v[0] != 99) { std::puts("write via operator[]"); return 1; }
    if (v.back() != 30) { std::puts("back wrong"); return 1; }
    if (!v.push_back(40)) { std::puts("fourth push_back"); return 1; }
    if (!v.full()) { std::puts("should be full"); return 1; }
    if (v.push_back(50)) { std::puts("push_back on full must return false"); return 1; }
    if (v.size() != 4) { std::puts("size after full-reject"); return 1; }
    if (sum_const(v) != 99 + 20 + 30 + 40) { std::puts("const sum wrong"); return 1; }
    v.clear();
    if (v.size() != 0 || v.full()) { std::puts("clear must reset size"); return 1; }
    if (!v.push_back(1)) { std::puts("reuse after clear"); return 1; }
    if (v.size() != 1 || v[0] != 1) { std::puts("state after reuse"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** An inline `small_vector` gives you `vector`'s ergonomics with `array`'s memory model: the elements are members, not a heap pointer, so the container is trivially copyable/relocatable and every access is a direct offset into a contiguous block that stays in cache. `push_back` refuses to grow rather than calling the allocator, which is exactly the trade you want on a latency-critical path where the maximum size is known and a mid-hot-path `malloc` (or a `vector` reallocation that invalidates iterators and copies every element) is unacceptable. Real libraries (LLVM's `SmallVector`, Boost's `small_vector`) generalize this with a heap fallback, but the fixed-capacity core here is the allocation-free hot-path building block.
