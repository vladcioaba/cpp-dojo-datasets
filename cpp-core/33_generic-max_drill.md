## exercise: Generic max
tags: templates, core

Write a function template `maxof` taking `class T` and two `const T&` parameters `a, b`, returning `const T&` — the larger of the two using `a < b ? b : a`.

hint: Write it once for any comparable type by making the type a template parameter.
hint: Take and return `const T&` and select with `a < b ? b : a`.

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

**Editorial:** A function template parameterized on `T` works for any type providing `operator<`, returning by `const T&` to avoid copies. Relying solely on `<` mirrors how the standard library expresses ordering. The drill teaches basic function templates and reference-returning. O(1).
