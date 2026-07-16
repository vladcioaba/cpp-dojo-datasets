## challenge: factorial computed by the compiler
tags: templates, nttp, constexpr
track: core
difficulty: easy

Write `factorial<N>()` — a `constexpr` function template whose only parameter is a non-type template parameter `unsigned N` — returning the factorial of `N` as `unsigned long long`. The result must be a constant expression: the harness checks it with `static_assert`, meaning the compiler itself does the multiplication. Recurse on `N` at compile time.

hint: The value travels in the template argument list, not the parameter list: `factorial<5>()`, no runtime arguments at all.
hint: A plain ternary `N == 0 ? 1 : N * factorial<N - 1>()` cannot terminate — instantiating `factorial<0>` would still instantiate `factorial<0 - 1>`, and `0u - 1` wraps to 4294967295. You need the recursion itself to stop at compile time.
hint: `if constexpr (N == 0) return 1; else return N * factorial<N - 1>();` — the discarded branch is never instantiated, so the template recursion bottoms out.

```cpp
// starter
// factorial<N>() -> unsigned long long, usable inside static_assert.
// N is a non-type template parameter (unsigned).
// TODO: write the constexpr function template.
```

```cpp
template <unsigned N>
constexpr unsigned long long factorial() {
    if constexpr (N == 0) {
        return 1;
    } else {
        return N * factorial<N - 1>();
    }
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    static_assert(factorial<0>() == 1);
    static_assert(factorial<1>() == 1);
    static_assert(factorial<5>() == 120);
    static_assert(factorial<10>() == 3628800ULL);
    static_assert(factorial<20>() == 2432902008176640000ULL);  // still fits in 64 bits
    assert(factorial<6>() == 720);                             // and callable at run time
    std::puts("PASS");
}
```

**Editorial:** Each `factorial<N>` is a distinct function stamped out at compile time, and because the function is `constexpr` with a constant `N`, the whole chain collapses into a single literal — `static_assert` proves no runtime work remains. The subtle part is termination: templates are instantiated *before* any runtime logic runs, so a regular `if` or ternary would still mention `factorial<N - 1>` in the `N == 0` instantiation, and unsigned wraparound sends `0 - 1` to `4294967295` — an instantiation avalanche that dies at the compiler's depth limit. `if constexpr` fixes this because the false branch of a discarded statement is not instantiated. Before C++17 the same problem was solved with a full specialization `template <> struct Factorial<0>` acting as the base case; `if constexpr` keeps base case and recursion in one readable body. `factorial<20>` is the largest that fits in 64 bits — one more multiplication would silently wrap in a non-constant context, but overflow in a constant expression is a compile error, another free safety net.
