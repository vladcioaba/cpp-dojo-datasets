## exercise: CRTP base
tags: patterns, crtp, templates

Write a struct template `Printable` taking `class D`, with a member function `print() const` that streams the derived object to `std::cout` via `std::cout << static_cast<const D&>(*this);`.

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
