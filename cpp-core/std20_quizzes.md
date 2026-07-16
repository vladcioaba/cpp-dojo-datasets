## quiz: What does this print?
tags: c++20, concepts
track: core
difficulty: medium

```cpp
template <class T>
concept HasSize = requires(T t) { t.size(); };

void report(HasSize auto const&) { std::cout << "sized "; }
void report(auto const&)         { std::cout << "plain "; }

int main() {
    report(std::string("hi"));
    report(42);
}
```

- [ ] Compile error — `42.size()` is ill-formed inside the requires expression
- [x] sized plain
- [ ] sized sized
- [ ] Compile error — the two overloads are ambiguous

> An unsatisfied constraint removes the overload from consideration — like SFINAE, it is a substitution failure, never a hard error, because the requires expression only *checks* whether `t.size()` would compile. For `std::string` both overloads are viable and the more-constrained one wins; for `int` only the unconstrained one survives. A hard error would occur only if the invalid expression appeared in the function *body*.

## quiz: How many times does the transform lambda run?
tags: c++20, ranges
track: core
difficulty: hard

```cpp
int calls = 0;
std::vector<int> v{1, 2, 3, 4, 5, 6};
auto r = v | std::views::filter([](int x) { return x % 2 == 0; })
           | std::views::transform([&](int x) { ++calls; return x * x; });
auto it = r.begin();
std::cout << *it << ' ' << calls;
```

- [ ] 4 0
- [x] 4 1
- [ ] 4 3
- [ ] 36 6

> Views are lazy: building the pipeline does no work at all. `r.begin()` makes `filter` scan forward to the first even element (2) but still doesn't call `transform`; only the dereference `*it` invokes it — once — producing 4. Nothing is ever computed for elements you don't visit.

## quiz: What does this print?
tags: c++20, spaceship
track: core
difficulty: easy

```cpp
struct Version {
    int major, minor;
    auto operator<=>(const Version&) const = default;
};

int main() {
    Version a{1, 2}, b{1, 10};
    std::cout << (a < b) << (a == b) << (a != b);
}
```

- [x] 101
- [ ] 011
- [ ] Compile error — defaulting <=> does not provide operator==
- [ ] 100

> A defaulted `<=>` compares members lexicographically in declaration order: majors tie, then 2 < 10, so `a < b` is true. Defaulting `operator<=>` also implicitly declares a defaulted `operator==`, so all six comparison operators work from this one line. (Equality is kept separate so types like `std::string` can short-circuit `==` on length.)

## quiz: What is the status of this initialization in standard C++20?
tags: c++20, designated-initializers
track: core
difficulty: medium

```cpp
struct Config { int width; int height; bool fullscreen; };

Config c{.height = 720, .width = 1280};
```

- [ ] Well-formed — same as C99 designated initializers
- [x] Ill-formed — designators must appear in declaration order
- [ ] Well-formed, but fullscreen is left uninitialized
- [ ] Undefined behavior

> Unlike C99, C++20 requires designators to follow the members' declaration order (`width` before `height`), which preserves the guarantee that initializers evaluate left-to-right in member order. GCC rejects this; Clang accepts it only as an extension with a warning. Reordered to `.width, .height` it is fine, and the omitted `fullscreen` would be value-initialized to `false`, not left uninitialized.

## quiz: What does this print?
tags: c++20, three-way-comparison
track: core
difficulty: hard

```cpp
double x = 1.0, y = std::nan("");
auto c = x <=> y;
std::cout << (c == std::partial_ordering::unordered)
          << (x < y) << (x > y) << (x == y);
```

- [ ] 0000
- [x] 1000
- [ ] 1001
- [ ] Compile error — <=> is not defined for double

> Floating-point `<=>` returns `std::partial_ordering` precisely because NaN is comparable to nothing: the result is `unordered` (first 1). In an unordered comparison, `<`, `>`, and `==` are all false (000). Integers instead get `strong_ordering`, where exactly one of less/equal/greater always holds — the return type documents the domain's guarantees.

## quiz: What is the behavior of this program?
tags: c++20, span
track: core
difficulty: medium

```cpp
std::vector<int> v{1, 2, 3, 4};
std::span<int> s(v);
v.push_back(5);
std::cout << s[0];
```

- [ ] Prints 1 — the span tracks the vector automatically
- [ ] Prints 5
- [x] Undefined behavior if push_back reallocated — s still points at the old buffer
- [ ] Compile error — span cannot view a vector

> `std::span` is a non-owning pointer+size over someone else's storage; it never registers with, or follows, the container it viewed. `push_back` may reallocate, freeing the buffer the span points into — after that, every access through `s` is UB. Same discipline as `string_view`: don't mutate or destroy the owner while a view is live.

## quiz: Which statement about constinit is true?
tags: c++20, constinit
track: core
difficulty: medium

```cpp
constexpr int default_port() { return 8080; }

constinit int port = default_port();

int main() { ++port; std::cout << port; }   // prints 8081
```

- [ ] It makes port immutable, like const
- [x] It guarantees static (compile-time) initialization, but the variable stays mutable
- [ ] It is a synonym for constexpr on variables
- [ ] It defers initialization until first use, like a function-local static

> `constinit` demands the initializer be a constant expression — so initialization happens at compile time, eliminating static-init-order fiasco races — but says nothing about the variable afterwards, which is why `++port` compiles. Swap the initializer for `std::rand()` and it fails to compile. `constexpr` on a variable implies both static initialization *and* const; `const` alone implies neither.

## quiz: In what order does this print?
tags: c++20, coroutines
track: core
difficulty: medium

Assume `generator<int>` is a typical lazy coroutine generator (its promise's `initial_suspend` returns `suspend_always`).

```cpp
generator<int> ticks() {
    std::cout << "start ";
    for (int i = 1;; ++i) co_yield i;
}

int main() {
    auto g = ticks();
    std::cout << "created ";
    std::cout << g.next();   // resumes the coroutine once
}
```

- [ ] start created 1
- [x] created start 1
- [ ] start 1 created
- [ ] It hangs — the infinite loop runs at the call site

> Calling a coroutine only allocates its frame and returns the generator object — the body is suspended at the initial suspend point, so "created" prints before "start". The first resume runs the body until `co_yield 1` suspends it back to the caller. The infinite loop is harmless: control returns to `main` at every yield, which is the whole point of a generator.

## quiz: What does this print?
tags: c++20, format
track: core
difficulty: easy

```cpp
std::cout << std::format("[{:>6.2f}] {:#x}", 3.14159, 255);
```

- [x] [  3.14] 0xff
- [ ] [3.14  ] 0xff
- [ ] [3.141590] 255
- [ ] Throws std::format_error at runtime

> `{:>6.2f}` means fixed notation, precision 2, right-aligned in width 6 — "3.14" padded with two leading spaces. `{:#x}` prints 255 in hex with the `0x` prefix. A malformed or arity-mismatched format string would not throw here: `std::format` checks literal format strings at **compile time**.

## quiz: What happens at the second call?
tags: c++20, template-lambdas
track: core
difficulty: easy

```cpp
auto add = []<typename T>(T a, T b) { return a + b; };

std::cout << add(1, 2);     // fine
std::cout << add(1, 2.5);   // ?
```

- [ ] Prints 3.5
- [ ] Prints 3 — the double is truncated
- [x] Compile error — T deduces both int and double
- [ ] Undefined behavior

> The C++20 template-parameter-list lambda forces both parameters to the **same** `T`, so `add(1, 2.5)` fails deduction: `T` cannot be `int` and `double` at once. The C++14 generic lambda `[](auto a, auto b)` would accept it, because each `auto` parameter is an independent template parameter — the same rule that makes `void f(auto x)` (an abbreviated function template) a real template.
