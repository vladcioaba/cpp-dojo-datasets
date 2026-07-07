## exercise: Visitor with std::variant
tags: patterns, visitor, variant

`v` is a `std::variant<int, std::string>`. Using `std::visit` and the `overloaded` idiom, return `std::size_t(x * 2)` for an `int x` and `s.size()` for a `const std::string& s` (both lambdas must return the same type — `std::visit` requires it). Assign the result to `auto n`.

hint: Dispatch on whichever alternative the variant currently holds, with a distinct handler per type.
hint: `std::visit` plus the `overloaded` idiom (a struct inheriting several lambdas) gives one branch per alternative.
hint: Every branch must return the same type, so wrap the int result in `std::size_t` to match `s.size()`.

```cpp
// starter
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
```

```cpp
auto n = std::visit(overloaded{
    [](int x) { return std::size_t(x * 2); },
    [](const std::string& s) { return s.size(); }
}, v);
```

```cpp
// harness
#include <variant>
#include <string>
#include <cstdio>
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
std::size_t run(std::variant<int, std::string> v) {
//__USER__
    return n;
}
int main() {
    if (run(21) != 42) return 1;
    if (run(std::string("hello")) != 5) return 1;
    std::puts("PASS");
}
```

**Editorial:** `std::visit` dispatches to the lambda that matches the variant's active alternative; the `overloaded` helper bundles the per-type lambdas into one callable via inheritance and a `using` declaration. All branches must yield a common return type, hence the `std::size_t` cast. The drill teaches type-safe visitation over a variant — the modern, hierarchy-free visitor.
