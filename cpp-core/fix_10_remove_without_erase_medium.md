## challenge: fix: the zeros that won't leave
tags: code-review, debugging, algorithms
track: core
difficulty: medium

This code review found a bug: after "stripping zeros" the vector has exactly the same size as before, and its tail contains stale leftover values. Find and fix it — keep the function signature.

hint: Check the vector's size after the call — did anything actually get removed?
hint: This is the remove/erase idiom with the second half missing.
hint: std::remove only shifts the kept elements to the front and returns the new logical end; it cannot change the vector's size, so the erase(...) call is mandatory.

```cpp
// starter
void stripZeros(std::vector<int>& v) {
    std::remove(v.begin(), v.end(), 0);
}
```

```cpp
void stripZeros(std::vector<int>& v) {
    v.erase(std::remove(v.begin(), v.end(), 0), v.end());
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> a{1, 0, 2, 0, 3};
    stripZeros(a);
    assert(a.size() == 3);                    // buggy: still 5
    assert(a == std::vector<int>({1, 2, 3}));

    std::vector<int> b{0, 0, 0};
    stripZeros(b);
    assert(b.empty());

    std::vector<int> c{4, 5};
    stripZeros(c);
    assert(c == std::vector<int>({4, 5}));

    std::puts("PASS");
}
```

**Editorial:** `std::remove` works on iterators and knows nothing about the container: it shifts every non-zero element toward the front and returns an iterator to the new logical end, leaving the physical size untouched and the tail full of unspecified leftovers (`{1,0,2,0,3}` becomes `{1,2,3,0,3}`). The removal is only complete when the container erases that tail — `v.erase(newEnd, v.end())` — which is why it is called the erase-remove idiom (C++20's `std::erase(v, 0)` does both steps). A reviewer spots this instantly: a call to `std::remove`/`remove_if` whose return value is discarded is always a bug.
