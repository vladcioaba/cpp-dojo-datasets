## exercise: Factory function
tags: patterns, factory

Write a function `make_circle` taking `double r` and returning a `std::unique_ptr<Shape>` holding a `Circle` constructed with `r`. Use `std::make_unique`. One line body.

```cpp
// starter
struct Shape { virtual ~Shape() = default; };
struct Circle : Shape { explicit Circle(double r); };
```

```cpp
std::unique_ptr<Shape> make_circle(double r) {
    return std::make_unique<Circle>(r);
}
```

```cpp
// harness
#include <memory>
#include <cstdio>
struct Shape { virtual ~Shape() = default; };
struct Circle : Shape { double r; explicit Circle(double r) : r(r) {} };
//__USER__
int main() {
    auto p = make_circle(2.5);
    if (!p) return 1;
    auto* c = dynamic_cast<Circle*>(p.get());
    if (!c || c->r != 2.5) return 1;
    std::puts("PASS");
}
```
