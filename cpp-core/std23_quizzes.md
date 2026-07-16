## quiz: What does this print?
tags: c++23, expected
track: core
difficulty: medium

```cpp
std::expected<int, std::string> parse(const std::string& s) {
    if (s.empty()) return std::unexpected("empty input");
    return static_cast<int>(s.size());
}

int main() {
    auto r = parse("").and_then([](int n) -> std::expected<int, std::string> {
        std::cout << "doubling ";
        return n * 2;
    });
    std::cout << (r ? std::to_string(*r) : r.error());
}
```

- [ ] doubling 0
- [x] empty input
- [ ] doubling empty input
- [ ] Throws an exception — the expected holds an error

> `and_then` runs its continuation only when the `expected` holds a value; on error it passes the error through untouched, so "doubling" never prints (`or_else` is the mirror image, running only on error). Nothing throws — unlike exceptions, the failure is an ordinary value in the return type, visible in the signature and dispatched with normal control flow.

## quiz: What does this print, and which C++23 feature makes it work?
tags: c++23, deducing-this
track: core
difficulty: medium

```cpp
auto fib = [](this auto self, int n) -> int {
    return n < 2 ? n : self(n - 1) + self(n - 2);
};
std::cout << fib(10);
```

- [x] 55 — the explicit object parameter lets the lambda name itself
- [ ] Compile error — a lambda cannot refer to itself
- [ ] 55 — but only because the lambda captures fib by reference
- [ ] Undefined behavior — self is dangling during recursion

> `this auto self` is C++23's explicit object parameter ("deducing this"): the closure object is passed as a visible, deduced parameter, so the body can simply call `self` — no `std::function` indirection or Y-combinator tricks. Nothing is captured; `fib(10)` is 55. The same mechanism lets one member function template replace the `const`/non-`const`/`&`/`&&` overload quadruplet and enables CRTP-style code without CRTP.

## quiz: What happens here?
tags: c++23, print
track: core
difficulty: easy

```cpp
#include <print>

int main() {
    std::println("{} + {} = {}", 1, 2);   // one argument short
}
```

- [ ] Prints "1 + 2 = "
- [ ] Throws std::format_error at runtime
- [x] Compile error — the format string is checked at compile time
- [ ] Undefined behavior

> `std::println` (like `std::print` and `std::format`) takes a `std::format_string`, whose constructor is `consteval`: the literal is parsed during compilation, and a third `{}` with only two arguments is rejected before the program ever runs. That is a categorical upgrade over `printf`, where the same mistake is runtime UB. `println` also appends a newline; `print` does not.

## quiz: What does this print?
tags: c++23, if-consteval
track: core
difficulty: medium

```cpp
constexpr int mode() {
    if consteval { return 1; }
    else         { return 2; }
}

int main() {
    constexpr int a = mode();
    int n = 0;
    int b = mode() + n;
    std::cout << a << b;
}
```

- [ ] 11
- [ ] 22
- [x] 12
- [ ] Compile error — mode() cannot behave differently at compile time

> `if consteval` asks "is this call happening during constant evaluation?" — initializing the `constexpr` variable takes the first branch (1), the plain runtime call takes the `else` (2). It fixes the classic `std::is_constant_evaluated()` trap of testing it inside `if constexpr` (always true) and, unlike the function, its consteval branch may call `consteval` functions.

## quiz: What does grid[1, 2] print?
tags: c++23, mdspan
track: core
difficulty: medium

```cpp
std::vector<int> data{1, 2, 3, 4, 5, 6};
std::mdspan grid(data.data(), 2, 3);   // 2 rows x 3 columns
std::cout << grid[1, 2];
```

- [ ] 5
- [x] 6
- [ ] Compile error — operator[] takes exactly one index
- [ ] 3

> `std::mdspan` is a non-owning multidimensional *view* over existing storage — it copies nothing and frees nothing. The default layout is row-major (`layout_right`), so `[1, 2]` maps to offset 1*3 + 2 = 5, the value 6. The multi-argument `operator[]` is itself a C++23 feature; C++20 would require the comma to be an operator inside the brackets.

## quiz: What does this print?
tags: c++23, ranges-to
track: core
difficulty: easy

```cpp
auto squares = std::views::iota(1, 5)
             | std::views::transform([](int x) { return x * x; })
             | std::ranges::to<std::vector>();
squares.push_back(25);
for (int x : squares) std::cout << x << ' ';
```

- [x] 1 4 9 16 25
- [ ] Compile error — you cannot push_back into a view
- [ ] 1 4 9 16 — the push_back is ignored because views are lazy
- [ ] 1 2 3 4 25

> `std::ranges::to<std::vector>` is the missing C++20 piece: it **eagerly** materializes the lazy pipeline into a real, owning container (element type deduced as `int`). `iota(1, 5)` is the half-open range 1..4, squared to 1 4 9 16. What comes out is an ordinary `std::vector`, so `push_back(25)` is perfectly fine — only the views themselves were immutable.

## quiz: Which statement about std::flat_map is true?
tags: c++23, flat-map
track: core
difficulty: hard

```cpp
std::flat_map<int, std::string> m{{3, "c"}, {1, "a"}, {2, "b"}};
// iterates 1 2 3 — sorted, like std::map
```

- [ ] It is a hash table, so lookups are O(1) on average
- [x] Lookup and iteration are cache-friendly thanks to contiguous sorted storage, but insertion is O(n) and invalidates iterators
- [ ] It behaves exactly like std::map, including stable iterators, just with a smaller header
- [ ] It keeps elements in insertion order to make inserts O(1)

> `flat_map` is a container adaptor over two sorted contiguous sequences (by default vectors of keys and of values): lookup is still O(log n) binary search but with cache-friendly probes, and iteration is a linear scan. The price is O(n) insert/erase (elements shift) and iterator invalidation on any insertion — the exact opposite of node-based `std::map`. Use it for build-once, query-many workloads; note its iterators hand out proxy references, so structured bindings need `const auto&` or by-value `auto`.

## quiz: What is the type of i?
tags: c++23, size-t-literal
track: core
difficulty: easy

```cpp
std::vector<int> v{10, 20, 30};
for (auto i = 0uz; i < v.size(); ++i)
    std::cout << v[i] << ' ';
```

- [ ] int
- [ ] unsigned int
- [x] std::size_t
- [ ] unsigned long long on every platform

> The C++23 `uz` suffix makes the literal a `std::size_t` (`z` alone gives the signed counterpart), so `auto` deduces exactly the type `v.size()` returns. That kills the signed/unsigned comparison warning of `int i = 0` without hardcoding a platform-dependent type — `size_t` is 32-bit on some targets, so "always unsigned long long" is wrong.

## quiz: What does std::stacktrace::current() capture?
tags: c++23, stacktrace
track: core
difficulty: medium

```cpp
void load_config(const std::string& path) {
    if (!exists(path))
        log_error("missing config", std::to_string(std::stacktrace::current()));
}
```

- [x] The calling thread's stack of function calls at the point where current() is invoked
- [ ] The stack as it was when the most recent exception was thrown
- [ ] A trace recorded at program startup
- [ ] Nothing unless called inside a catch block

> `std::stacktrace::current()` snapshots the current thread's call sequence right where you call it — `load_config`, its caller, and so on up to `main`. That is why you capture it where the error is *detected* (or store one inside an exception type), not in the `catch` handler, where unwinding has already erased the interesting frames. Each `stacktrace_entry` exposes `description()`, `source_file()`, and `source_line()` for logging.

## quiz: Which statement about [[assume]] is true?
tags: c++23, assume
track: core
difficulty: hard

```cpp
int bucket(int x) {
    [[assume(x >= 0)]];
    return x / 16;
}
```

- [ ] The expression is evaluated at runtime and aborts if false, like assert
- [ ] It is a compile error unless the compiler can prove x >= 0
- [x] The expression is never evaluated; the optimizer may rely on it, and calling bucket(-5) is undefined behavior
- [ ] It is purely documentation with no effect on code generation

> `[[assume]]` hands the optimizer a fact it may exploit without checking it: here signed `x / 16` can compile to a plain arithmetic shift, dropping the negative-rounding fix-up. The expression is *not* evaluated — it only had to be plausible — and if it would be false at runtime, behavior is undefined. Think of it as the inverse of `assert`: assert verifies and never optimizes, assume optimizes and never verifies.
