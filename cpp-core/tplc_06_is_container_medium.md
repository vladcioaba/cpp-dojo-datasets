## challenge: is it a container? ask a concept
tags: templates, concepts
track: core
difficulty: medium

Write a concept `Container` that holds when a `const T&` supports `t.begin()` and `t.end()`. Then write two overloads of `describe`, selected by requires-clauses: for containers, return `"container[N]"` where `N` is the element count (`std::to_string`); for everything else, return `"scalar"`. Both overloads take `const T&`.

hint: A `requires` expression lists the expressions that must compile: `requires(const T& t) { t.begin(); t.end(); }` — that whole thing is the concept's definition.
hint: Two function templates with the same signature can coexist if their requires-clauses are mutually exclusive: one `requires Container<T>`, the other `requires (!Container<T>)`.
hint: Count elements without assuming `.size()` exists: `std::distance(value.begin(), value.end())` works for anything with begin/end.

```cpp
// starter
// concept Container: t.begin() and t.end() valid on a const T&.
// describe(x) -> "container[N]" for containers, "scalar" otherwise.
// TODO: define the concept, then the two requires-constrained overloads.
```

```cpp
template <class T>
concept Container = requires(const T& t) {
    t.begin();
    t.end();
};

template <class T>
    requires Container<T>
std::string describe(const T& value) {
    auto n = std::distance(value.begin(), value.end());
    return "container[" + std::to_string(n) + "]";
}

template <class T>
    requires (!Container<T>)
std::string describe(const T&) {
    return "scalar";
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> v{1, 2, 3, 4};
    std::list<double> l{1.5, 2.5};
    std::map<int, int> m{{1, 1}};
    assert(describe(v) == "container[4]");
    assert(describe(l) == "container[2]");
    assert(describe(m) == "container[1]");
    assert(describe(42) == "scalar");
    assert(describe(3.14) == "scalar");
    assert(describe(std::string("abc")) == "container[3]");  // begin/end exist, so it counts
    static_assert(Container<std::vector<int>>);
    static_assert(!Container<int>);
    std::puts("PASS");
}
```

**Editorial:** The concept is duck typing made explicit: `Container<T>` is true exactly when the two expressions inside the `requires` block would compile for a `const T&` — no registration, no inheritance, and the trait works for any past or future type with the right shape. Probing on `const T&` matters: it means detection requires const-callable `begin()`/`end()`, which every real container provides. The two overloads are distinguished *only* by their requires-clauses; because `Container<T>` and `!Container<T>` can never both hold, exactly one candidate survives overload resolution for any argument — the C++20 replacement for `enable_if` pairs. Note the honest surprise in the harness: `std::string` has `begin`/`end`, so this trait calls it a container — structural detection answers the question you actually asked, not the one you had in mind. Counting with `std::distance` instead of `.size()` keeps the requirement set minimal.
