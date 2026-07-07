## fact: if constexpr killed half of template metaprogramming
tags: templates, core

Before C++17, choosing behavior per-type inside a template needed tag dispatch or SFINAE overloads. `if constexpr` discards the untaken branch at compile time — the discarded branch doesn't even need to compile for that type.

```cpp
template <class T>
auto describe(const T& x) {
    if constexpr (std::is_integral_v<T>)
        return x * 2;              // only compiled for ints
    else
        return x.size();           // only compiled for containers
}
```

With C++20 concepts, `requires` clauses handle the overload-selection half too.
