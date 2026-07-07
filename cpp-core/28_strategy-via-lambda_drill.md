## exercise: Strategy via lambda
tags: patterns, strategy

Sort `v` (a `std::vector<int>`) in **descending** order using `std::sort` and a lambda taking `int a, int b`. One statement.

hint: The comparison is a pluggable strategy — pass it in rather than hard-coding the order.
hint: Hand `std::sort` a lambda `(int a, int b){ return a > b; }` to sort descending.

```cpp
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

```cpp
// harness
#include <algorithm>
#include <vector>
#include <cstdio>
int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};
//__USER__
    if (!std::is_sorted(v.begin(), v.end(), [](int a, int b) { return a > b; })) return 1;
    std::puts("PASS");
}
```

**Editorial:** The strategy pattern here is just a comparator lambda handed to `std::sort`; returning `a > b` flips the ordering to descending. In modern C++ a small callable replaces an entire strategy class hierarchy. The drill teaches passing behavior as a lambda; `std::sort` runs in O(n log n).
