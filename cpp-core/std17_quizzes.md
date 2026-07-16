## quiz: What does this print?
tags: c++17, structured-bindings
track: core
difficulty: easy

```cpp
std::pair<int, int> p{1, 2};
auto [a, b] = p;
a = 10;
std::cout << p.first << ' ' << a;
```

- [ ] 10 10
- [x] 1 10
- [ ] 1 1
- [ ] Compile error — bindings are read-only

> `auto [a, b] = p` first copies `p` into a hidden object; the names bind to pieces of that **copy**, so writing `a` never touches `p`. Use `auto& [a, b] = p` to bind references into the original. Note the bindings themselves are names, not separate variables — `decltype(a)` is `int`, not `int&`.

## quiz: What does this print?
tags: c++17, if-init
track: core
difficulty: easy

```cpp
std::map<std::string, int> m{{"a", 1}};
if (auto it = m.find("b"); it != m.end())
    std::cout << it->second;
else
    std::cout << "miss " << (it == m.end());
```

- [ ] Compile error — `it` is out of scope in the else branch
- [ ] 1
- [x] miss 1
- [ ] miss 0

> The variable declared in the if-statement's init clause is in scope for **both** branches, so the else branch can still compare `it` (and it equals `end()`, printing 1). The lookup misses, so control goes to else. This pattern keeps `it` from leaking into the surrounding scope.

## quiz: What happens when this compiles?
tags: c++17, if-constexpr
track: core
difficulty: medium

Note the plain `if` — not `if constexpr`.

```cpp
template <class T>
auto describe(T x) {
    if (std::is_integral_v<T>)
        return (unsigned long)(x * 2);
    else
        return (unsigned long)x.size();
}
int main() { std::cout << describe(21); }
```

- [ ] Prints 42
- [x] Compile error — `int` has no member `size()`
- [ ] Prints garbage
- [ ] Runtime error in the else branch

> A plain `if` instantiates **both** branches for every `T`, and `21.size()` is ill-formed. With `if constexpr` the false branch is discarded per instantiation and this prints 42. Unlike `#ifdef`, `if constexpr` still requires both branches to parse and works on template parameters, not preprocessor symbols.

## quiz: What is the behavior of this program?
tags: c++17, optional
track: core
difficulty: medium

```cpp
std::optional<int> port;   // empty
std::cout << port.value_or(8080) << ' ';
std::cout << *port;
```

- [ ] Prints "8080 0"
- [ ] Prints 8080, then `*port` throws std::bad_optional_access
- [x] Prints 8080, then undefined behavior
- [ ] Compile error — cannot dereference an optional

> `value_or` returns the fallback when empty — that part is safe. But `operator*` on an empty optional is **undefined behavior**, not an exception; it is `value()` that throws `bad_optional_access`. In practice you get garbage or a crash, so check `has_value()` or prefer `value_or`/`value()`.

## quiz: What does this print?
tags: c++17, variant
track: core
difficulty: easy

```cpp
std::variant<int, std::string> v = 7;
v = "hi";
std::visit([](auto&& x) { std::cout << x; }, v);
std::cout << ' ' << v.index();
```

- [x] hi 1
- [ ] 7 0
- [ ] hi 0
- [ ] Compile error — the visitor must handle each alternative separately

> Assignment switches the active alternative to `std::string` (index 1, the second type), and a generic lambda is a valid visitor for all alternatives. `std::visit` dispatches on the runtime index. If a type-changing assignment throws mid-switch the variant can end up `valueless_by_exception()` — then `visit` throws `bad_variant_access`.

## quiz: What is the behavior of this program?
tags: c++17, string-view
track: core
difficulty: medium

```cpp
std::string make() { return "hello world"; }

int main() {
    std::string_view sv = make();
    std::cout << sv;
}
```

- [ ] Prints "hello world"
- [ ] Prints an empty string
- [x] Undefined behavior — sv dangles
- [ ] Compile error — string_view cannot bind to a temporary

> `string_view` is a non-owning pointer+length. The temporary `std::string` returned by `make()` is destroyed at the end of the declaration's full-expression, leaving `sv` pointing into freed memory. It often *appears* to work, which is exactly why this bug ships. Never bind a `string_view` to a temporary you don't outlive.

## quiz: What does the vector contain?
tags: c++17, ctad
track: core
difficulty: medium

```cpp
std::vector v{3, 5};
std::cout << v.size() << ' ' << v[0] << ' ' << v[1];
```

- [x] 2 3 5 — two elements, 3 and 5
- [ ] 3 5 5 — three elements, all 5
- [ ] Compile error — vector needs an explicit element type
- [ ] 2 0 0 — two value-initialized elements

> Class template argument deduction infers `std::vector<int>` from the initializers, and braces select the `initializer_list` constructor: two elements, 3 and 5. The classic trap is the parenthesized form — `std::vector<int> v(3, 5)` means "three copies of 5". CTAD only picks the template arguments; it doesn't change which constructor braces prefer.

## quiz: What happens when this program is linked?
tags: c++17, inline-variables
track: core
difficulty: medium

```cpp
// config.h — included by both a.cpp and b.cpp
inline int hit_count = 0;

// a.cpp
void bump() { ++hit_count; }

// b.cpp
int main() { bump(); ++hit_count; std::cout << hit_count; }
```

- [ ] Linker error — duplicate symbol hit_count
- [ ] Prints 1 — each translation unit gets its own copy
- [x] Prints 2 — all translation units share one variable
- [ ] Undefined behavior — ODR violation

> A C++17 `inline` variable may be defined identically in multiple translation units; the linker merges them into **one** entity, so both increments hit the same object and it prints 2. Without `inline`, defining it in a header is an ODR violation (duplicate-symbol link error). Pre-C++17 you needed an `extern` declaration plus exactly one out-of-line definition.

## quiz: What does the standard say happens here?
tags: c++17, nodiscard
track: core
difficulty: easy

```cpp
[[nodiscard]] int connect() { return -1; }

int main() { connect(); }
```

- [ ] Ill-formed — the result must be used
- [x] It compiles; implementations are encouraged to warn about the ignored result
- [ ] std::terminate is called at runtime
- [ ] Undefined behavior

> `[[nodiscard]]` makes discarding the return value diagnosable, not ill-formed: compilers emit a warning (an error only under flags like `-Werror`) and the program runs normally. Cast to `void` to silence it deliberately. `[[maybe_unused]]` is the mirror-image attribute — it suppresses unused-entity warnings.

## quiz: The comparator throws during a parallel sort. What happens?
tags: c++17, parallel-algorithms
track: core
difficulty: hard

```cpp
std::sort(std::execution::par, v.begin(), v.end(),
          [](int a, int b) {
              if (a == INT_MIN) throw std::runtime_error("bad");
              return a < b;
          });
```

- [ ] The exception propagates to the caller of std::sort
- [ ] The exception is stored and rethrown after the algorithm finishes
- [x] std::terminate is called
- [ ] The algorithm silently falls back to sequential execution

> If an element access function (comparator, predicate, projection) exits via an exception under **any** execution policy, the standard requires `std::terminate` — there is no defined way to unwind across worker threads. The other parallel contract is on you: callbacks must be data-race free. Only `bad_alloc` from the algorithm's own internal allocation may propagate.
