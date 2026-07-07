## exercise: CRTP base
tags: patterns, crtp, templates

Write a struct template `Printable` taking `class D`, with a member function `print() const` that streams the derived object to `std::cout` via `std::cout << static_cast<const D&>(*this);`.

hint: The base can call into the derived class without a vtable if it knows the derived type statically.
hint: Template the base on the derived type and `static_cast<const D&>(*this)` to reach the derived object.

```cpp
template <class D>
struct Printable {
    void print() const {
        std::cout << static_cast<const D&>(*this);
    }
};
```

```cpp
// harness
#include <iostream>
#include <sstream>
#include <cstdio>
//__USER__
struct Point : Printable<Point> {
    int x = 7;
};
std::ostream& operator<<(std::ostream& os, const Point& p) { return os << "P" << p.x; }
int main() {
    std::ostringstream out;
    auto* old = std::cout.rdbuf(out.rdbuf());
    Point{}.print();
    std::cout.rdbuf(old);
    if (out.str() != "P7") return 1;
    std::puts("PASS");
}
```

**Editorial:** The Curiously Recurring Template Pattern passes the derived type as a template parameter, so the base can `static_cast` `*this` to `D&` and call derived behavior with everything resolved at compile time — polymorphism at zero virtual-dispatch cost. The drill teaches static (compile-time) polymorphism via CRTP.
