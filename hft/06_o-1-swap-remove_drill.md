## challenge: O(1) swap-remove
tags: cache, vector, data-structures
track: hft

When order doesn't matter, erasing from the middle of a `std::vector` in O(n) (shifting) is wasteful. Swap the target with the last element and pop — O(1), cache-friendly. Implement `void swap_remove(std::vector<int>& v, size_t i)` removing the element at index `i` (assume `i < v.size()`).

```cpp
void swap_remove(std::vector<int>& v, size_t i) {
    v[i] = v.back();
    v.pop_back();
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <cstddef>
//__USER__
int main() {
    std::vector<int> v{10, 20, 30, 40, 50};
    swap_remove(v, 1);                 // remove 20 -> {10,50,30,40}
    if (v.size() != 4) { std::puts("size wrong"); return 1; }
    if (v[0]!=10 || v[1]!=50 || v[2]!=30 || v[3]!=40) { std::puts("contents wrong"); return 1; }
    swap_remove(v, 3);                 // remove last (40) -> {10,50,30}
    if (v.size()!=3 || v[2]!=30) { std::puts("remove-last wrong"); return 1; }
    swap_remove(v, 0);                 // remove 10 -> {30,50}
    if (v.size()!=2 || v[0]!=30 || v[1]!=50) { std::puts("remove-first wrong"); return 1; }
    std::puts("PASS");
}
```
