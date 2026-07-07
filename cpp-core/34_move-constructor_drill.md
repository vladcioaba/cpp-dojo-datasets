## exercise: Move constructor
tags: move, core

`Buffer` owns `int* data` and `size_t n`. Write its move constructor: take `Buffer&& other` (noexcept), steal `data` and `n` via member-init list `data(other.data), n(other.n)`, then null out the source: `other.data = nullptr; other.n = 0;`.

hint: Moving means stealing the source's resources rather than copying them, then leaving the source safe to destroy.
hint: Copy the pointer and size in the initializer list, then null out the source's pointer.
hint: Mark it `noexcept` so containers like `std::vector` will move rather than copy during reallocation.

```cpp
// starter
class Buffer {
    int* data; size_t n;
public:
    // your code here
};
```

```cpp
Buffer(Buffer&& other) noexcept : data(other.data), n(other.n) {
    other.data = nullptr;
    other.n = 0;
}
```

```cpp
// harness
#include <cstddef>
#include <cstdio>
#include <utility>
class Buffer {
public:
    int* data; size_t n;
    Buffer(size_t k) : data(new int[k]), n(k) {}
    ~Buffer() { delete[] data; }
//__USER__
};
int main() {
    Buffer a(5);
    int* p = a.data;
    Buffer b(std::move(a));
    if (b.data != p || b.n != 5) return 1;
    if (a.data != nullptr || a.n != 0) return 1;
    std::puts("PASS");
}
```

**Editorial:** The move constructor transfers ownership by copying the pointer and size, then resetting the source (`data = nullptr`) so its destructor will not double-free. Marking it `noexcept` lets containers move instead of copy when they reallocate. The drill teaches move semantics and the leave-the-source-valid rule. O(1).
