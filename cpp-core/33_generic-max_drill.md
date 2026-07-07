## exercise: Generic max
tags: templates, core

Write a function template `maxof` taking `class T` and two `const T&` parameters `a, b`, returning `const T&` — the larger of the two using `a < b ? b : a`.

```cpp
template <class T>
const T& maxof(const T& a, const T& b) {
    return a < b ? b : a;
}
```

```cpp
// harness
#include <string>
#include <cstdio>
//__USER__
int main() {
    if (maxof(2, 3) != 3) return 1;
    if (maxof(std::string("apple"), std::string("banana")) != "banana") return 1;
    std::puts("PASS");
}
```
