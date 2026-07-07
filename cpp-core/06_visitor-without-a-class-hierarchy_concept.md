## fact: Visitor without a class hierarchy
tags: patterns, visitor, variant

`std::variant` + `std::visit` + the `overloaded` idiom replaces the classic double-dispatch Visitor pattern — no base class, no `accept()` boilerplate, and the compiler errors if you forget a case.

```cpp
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };

std::variant<Circle, Square> shape = Circle{2.0};
double a = std::visit(overloaded{
    [](const Circle& c) { return 3.14159 * c.r * c.r; },
    [](const Square& s) { return s.side * s.side; }
}, shape);
```
