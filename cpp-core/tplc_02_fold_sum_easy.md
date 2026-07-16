## challenge: sum any number of arguments with one fold
tags: templates, variadic
track: core
difficulty: easy

Write a variadic function template `sumAll` that returns the sum of all its arguments using a fold expression — no recursion, no loops. The empty call `sumAll()` must compile and return `0`, and the function must be `constexpr` so results can be checked at compile time.

hint: Declare a parameter pack (`template <class... Ts>`, parameters `Ts... args`) and expand it with a fold expression instead of writing a recursive helper.
hint: A unary fold `(args + ...)` is ill-formed for an empty pack — there is a fold form that supplies an initial value.
hint: `return (args + ... + 0);` — the binary fold seeds the sum with `0`, which both handles `sumAll()` and keeps the whole thing one expression.

```cpp
// starter
// Sum every argument with a single fold expression.
// sumAll() with no arguments must return 0. Keep it constexpr.
// TODO: write the variadic function template.
```

```cpp
template <class... Ts>
constexpr auto sumAll(Ts... args) {
    return (args + ... + 0);
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    static_assert(sumAll() == 0);              // empty pack folds to the seed
    static_assert(sumAll(7) == 7);
    static_assert(sumAll(1, 2, 3, 4) == 10);
    assert(sumAll(1.5, 2.25) == 3.75);
    assert(sumAll(1, 2.5) == 3.5);             // mixed pack: usual arithmetic promotions
    assert(sumAll(1u, 2u, 3u) == 6u);
    std::puts("PASS");
}
```

**Editorial:** `(args + ... + 0)` is a binary right fold: for `sumAll(1, 2, 3)` it expands to `1 + (2 + (3 + 0))`. The seed value earns its keep twice — a unary `+` fold over an empty pack is ill-formed by rule, so `(args + ...)` would reject `sumAll()`, while the binary form simply yields the seed. Each element keeps its own type during expansion, so mixed packs follow the ordinary arithmetic promotion rules (`1 + 2.5` becomes `double`), and `auto` deduces the final result type. Before C++17 this function needed a recursive peel-one-off overload plus a base case; the fold collapses all of that into one line that the optimizer sees straight through. Marking it `constexpr` costs nothing and lets callers verify sums with `static_assert`.
