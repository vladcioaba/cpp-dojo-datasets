## exercise: Strategy via lambda
tags: patterns, strategy

Sort `v` (a `std::vector<int>`) in **descending** order using `std::sort` and a lambda taking `int a, int b`. One statement.

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
