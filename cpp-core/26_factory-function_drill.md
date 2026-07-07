## exercise: Factory function
tags: patterns, factory

Write a function `make_circle` taking `double r` and returning a `std::unique_ptr<Shape>` holding a `Circle` constructed with `r`. Use `std::make_unique`. One line body.

hint: Hand back ownership of a base-class pointer while actually constructing a concrete derived type.
hint: `std::make_unique<Circle>(r)` constructs the concrete type and implicitly converts to `unique_ptr<Shape>` on return.

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

**Editorial:** A factory returns a `unique_ptr<Shape>` while constructing a concrete `Circle`, hiding the concrete type behind the base interface and transferring ownership safely. `std::make_unique` allocates and constructs in one exception-safe step. The drill teaches factory functions built on smart-pointer ownership.
