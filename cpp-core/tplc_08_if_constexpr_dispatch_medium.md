## challenge: one template, three formats
tags: templates, if-constexpr
track: core
difficulty: medium

Write a function template `stringify(value)` returning `std::string`, dispatching on the type at compile time with `if constexpr`: for integral types return `"int:" + std::to_string(value)`; for floating-point types return `"real:" + std::to_string(value)`; for anything else assume it is string-like and return `"str:"` followed by the value. A runtime `if` cannot work here — figure out why before reaching for the hints.

hint: Test the type, not the value: `std::is_integral_v<T>` and `std::is_floating_point_v<T>` from `<type_traits>` are compile-time answers.
hint: With a plain `if`, every branch must compile for every `T` — and `std::to_string(std::string)` does not exist, nor does `"str:" + 42`. The dispatch has to *remove* branches, not skip them.
hint: `if constexpr (std::is_integral_v<T>) ... else if constexpr (std::is_floating_point_v<T>) ... else ...` — discarded branches are never instantiated, so each `T` only compiles the code that makes sense for it.

```cpp
// starter
// stringify(42)                  -> "int:42"
// stringify(2.5)                 -> "real:2.500000"
// stringify(std::string("abc"))  -> "str:abc"
// TODO: one function template, branches selected at compile time.
template <class T>
std::string stringify(const T& value) {
    return {}; // TODO
}
```

```cpp
template <class T>
std::string stringify(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        return "int:" + std::to_string(value);
    } else if constexpr (std::is_floating_point_v<T>) {
        return "real:" + std::to_string(value);
    } else {
        return std::string("str:") + value;
    }
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(stringify(42) == "int:42");
    assert(stringify(-7L) == "int:-7");
    assert(stringify(true) == "int:1");            // bool is integral
    assert(stringify(2.5) == "real:2.500000");     // std::to_string uses %f
    assert(stringify(0.5f) == "real:0.500000");
    assert(stringify(std::string("abc")) == "str:abc");
    assert(stringify("lit") == "str:lit");         // char array lands in the else branch
    std::puts("PASS");
}
```

**Editorial:** The whole point is *discarded statements*: inside a template, the false branches of `if constexpr` are not instantiated, so `std::to_string(value)` is only ever compiled when `T` is arithmetic, and `std::string("str:") + value` only when it isn't. Swap in a runtime `if` and the function stops compiling for every type, because all three branches would have to type-check simultaneously — `std::to_string` has no overload for `std::string`, and `"str:" + 42` is pointer arithmetic, not concatenation. This is the C++17 idiom that replaced tag dispatch and heaps of SFINAE overloads with straight-line code. Two harness details worth noticing: `bool` is an integral type, so `true` prints as `int:1` (add an `is_same_v<T, bool>` branch first if you want `bool` treated specially), and a string literal arrives as `const char[4]`, is neither integral nor floating, and decays to `const char*` in the concatenation — the else branch quietly handles both string flavors.
