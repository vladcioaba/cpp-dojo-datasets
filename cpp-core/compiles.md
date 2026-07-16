# Does this compile?

25 cards on the compile/UB/runtime boundary: every snippet is a complete program — decide whether it compiles, and if it does, what happens.

## quiz: Does std::max(1, 2.5) compile?
tags: compilation, templates, deduction
track: core
difficulty: easy

A helper picks the larger of two numbers of different types.

```cpp
#include <algorithm>
int main() {
    int small = 1;
    double big = 2.5;
    return std::max(small, big);
}
```

- [ ] Compiles and returns 2
- [ ] Compiles; the int is promoted to double before the call
- [x] Compile error: template argument deduction fails
- [ ] Compiles, but comparing int and double is undefined behavior

> `std::max` is `template <class T> const T& max(const T&, const T&)`. Deduction from the first argument says `T = int`, from the second `T = double` — conflicting deductions make the call ill-formed. Deduction must produce one consistent `T` before any conversions are considered. Fix: `std::max<double>(small, big)` or cast one argument.

## quiz: Dependent type without typename
tags: compilation, templates, typename
track: core
difficulty: medium

A generic function grabs an iterator to the first element.

```cpp
#include <vector>
template <typename C>
void first_of(const C& c) {
    C::const_iterator it = c.begin();
    (void)it;
}
int main() { std::vector<int> v{1}; first_of(v); }
```

- [x] Compile error: `C::const_iterator` needs `typename`
- [ ] Compiles and runs fine
- [ ] Compile error: `std::vector<int>` has no `const_iterator`
- [ ] Compiles, but only when called with a non-const container

> `C::const_iterator` is a dependent name, and in a block-scope declaration the compiler must be told it names a type: `typename C::const_iterator it = ...`. C++20 dropped the `typename` requirement in a few contexts where only a type is grammatically possible (return types, member declarations), but a declaration inside a function body is not one of them.

## quiz: Same inline function defined twice in one file
tags: compilation, odr, inline
track: core
difficulty: medium

The same inline function appears twice, token for token.

```cpp
inline int answer() { return 42; }
inline int answer() { return 42; }
int main() { return answer(); }
```

- [ ] Compiles: inline allows repeated identical definitions
- [x] Compile error: redefinition within one translation unit
- [ ] Compiles, but which definition is used is unspecified
- [ ] Links, but with undefined behavior

> `inline` relaxes the one-definition rule *across* translation units — each TU that uses the function must contain its own identical definition. Within a single translation unit the normal rule still applies: one definition only, so the second one is a plain redefinition error.

## quiz: int argument, long and double overloads
tags: compilation, overloading, conversions
track: core
difficulty: easy

Two overloads exist; the caller passes a plain int.

```cpp
#include <iostream>
void report(long x)   { std::cout << "long"; }
void report(double x) { std::cout << "double"; }
int main() { report(7); }
```

- [ ] Prints long: the integral overload is preferred
- [ ] Prints double
- [x] Compile error: the call is ambiguous
- [ ] Prints long: it is declared first

> `int → long` and `int → double` are both conversion-rank standard conversions (`int → long` is a conversion, not a promotion — only e.g. `short`/`char → int` are promotions). Neither overload is better than the other, so overload resolution fails and the call is ill-formed. Declaration order never breaks ties.

## quiz: String literal vs a bool overload
tags: conversions, overloading
track: core
difficulty: medium

A logging helper is overloaded for messages and for flags.

```cpp
#include <iostream>
#include <string>
void log_msg(const std::string& s) { std::cout << "string"; }
void log_msg(bool b)               { std::cout << "bool"; }
int main() { log_msg("disk full"); }
```

- [ ] Prints string
- [x] Prints bool
- [ ] Compile error: the call is ambiguous
- [ ] Compile error: a string literal cannot convert to bool

> `const char*` → `bool` is a standard conversion (pointer to bool), while `const char*` → `std::string` needs a user-defined conversion. Standard conversions always outrank user-defined ones in overload resolution, so the `bool` overload wins silently. This is a real API-design trap: never overload on `bool` next to string-like parameters.

## quiz: Pass-by-value with a deleted copy constructor
tags: compilation, special-members
track: core
difficulty: easy

Socket forbids copying; transmit takes it by value.

```cpp
struct Socket {
    Socket() = default;
    Socket(const Socket&) = delete;
};
void transmit(Socket s) {}
int main() {
    Socket s;
    transmit(s);
}
```

- [x] Compile error at the call `transmit(s)`
- [ ] Compiles: pass-by-value falls back to the move constructor
- [ ] Compile error at the definition of `transmit`
- [ ] Compiles, but s is left in a moved-from state

> Passing an lvalue by value copy-initializes the parameter, which selects the copy constructor. Deleted functions still participate in overload resolution — being chosen is what makes the program ill-formed, at the call site. Declaring the copy constructor (even as deleted) also suppresses the implicit move constructor, so there is no move fallback; the function definition itself is fine.

## quiz: Returning a reference to a local
tags: ub, lifetime, references
track: core
difficulty: medium

biggest returns a reference to avoid a copy.

```cpp
int& biggest(int a, int b) {
    int result = a > b ? a : b;
    return result;
}
int main() {
    int& r = biggest(1, 2);
    return r;
}
```

- [ ] Compile error: a local cannot be returned by reference
- [ ] Compiles and reliably returns 2
- [x] Compiles, but reading r is undefined behavior
- [ ] Compiles; the reference keeps result alive

> `result` dies when `biggest` returns, so the returned reference dangles. The language still accepts the code — compilers emit at most a warning — and the UB happens when `r` is read in `main`. Lifetime extension applies only to temporaries bound directly to references, never through a `return`; return by value here.

## quiz: constexpr division by zero
tags: compilation, constexpr, ub
track: core
difficulty: medium

A constexpr helper is asked to divide by zero at compile time.

```cpp
constexpr int safe_div(int a, int b) { return a / b; }
int main() {
    constexpr int x = safe_div(10, 0);
    return x;
}
```

- [ ] Compiles; x is 0
- [ ] Compiles, but crashes at run time
- [x] Compile error: the initializer is not a constant expression
- [ ] Undefined behavior at run time

> Constant evaluation is not allowed to encounter undefined behavior. `constexpr int x` forces compile-time evaluation of `safe_div(10, 0)`, and the division by zero disqualifies it as a constant expression, so the program is ill-formed. The same call stored in a plain `int` would compile — and be UB at run time. constexpr turns a class of UB into diagnostics.

## quiz: Three names for a two-element pair
tags: compilation, structured-bindings
track: core
difficulty: medium

A pair is decomposed with a structured binding.

```cpp
#include <utility>
int main() {
    std::pair<int, int> p{1, 2};
    auto [x, y, z] = p;
    return x + y + z;
}
```

- [ ] Compiles; z is value-initialized to 0
- [x] Compile error: the name count must match the element count exactly
- [ ] Compiles; z aliases y
- [ ] Compile error: std::pair does not support structured bindings

> A structured binding must introduce exactly as many names as the type decomposes into — `std::tuple_size<std::pair<int,int>>::value` is 2, and three names is ill-formed. There is no padding, defaulting, or truncation. (C++26 adds `_` placeholders for *ignoring* elements, but never for inventing extra ones.)

## quiz: Lambda reads a local it never captured
tags: compilation, lambdas
track: core
difficulty: easy

A predicate lambda uses a local variable with an empty capture list.

```cpp
int main() {
    int limit = 10;
    auto over = [](int x) { return x > limit; };
    return over(5);
}
```

- [x] Compile error: limit is not captured
- [ ] Compiles; locals are captured by value automatically
- [ ] Compiles; lambdas may read (but not write) enclosing locals
- [ ] Compiles, but reads an indeterminate value

> A lambda body may only name local variables of automatic storage duration that it captures — explicitly (`[limit]`) or via a capture-default (`[=]`, `[&]`). An empty `[]` captures nothing, so `limit` is not reachable and the program is ill-formed. Globals, statics, and constant expressions are usable without capture; plain locals are not.

## quiz: Braced init from a double expression
tags: compilation, initialization, narrowing
track: core
difficulty: easy

A percentage is computed and stored via list-initialization.

```cpp
int main() {
    double ratio = 0.5;
    int percent{ratio * 100};
    return percent;
}
```

- [ ] Compiles; percent is 50
- [x] Compile error: narrowing conversion in braced initialization
- [ ] Compiles, truncating only if the value does not fit in int
- [ ] Compiles; braces behave exactly like = here

> List-initialization forbids narrowing conversions, and `double → int` narrows unless the value is a constant expression provably representable — `ratio * 100` is a runtime value, so it is ill-formed. `int percent = ratio * 100;` would compile and silently truncate. Turning that silent truncation into an error is precisely what braces are for.

## quiz: Non-const call through a const reference
tags: compilation, const
track: core
difficulty: easy

inspect takes the counter by const reference — and pokes it.

```cpp
struct Counter {
    int n = 0;
    void bump() { ++n; }
};
void inspect(const Counter& c) { c.bump(); }
int main() { Counter c; inspect(c); }
```

- [x] Compile error: bump() is not a const member function
- [ ] Compiles; const on a reference parameter is advisory
- [ ] Compiles; members of a local struct are mutable by default
- [ ] Compile error: a const object cannot be passed by reference

> Through a `const Counter&`, only `const` member functions may be called; `bump()` is non-const because it modifies `n`, so the call is ill-formed. The `this` pointer inside a const call is `const Counter*`, and the compiler enforces it. The fixes — non-const parameter, or `mutable n` — are each a different design statement, which is why const-correctness propagates through APIs.

## quiz: Virtual function, different default arguments
tags: virtual, default-arguments
track: core
difficulty: hard

Base and Derived both default the who parameter — differently.

```cpp
#include <iostream>
struct Base {
    virtual void greet(const char* who = "Base") { std::cout << "Base::" << who; }
};
struct Derived : Base {
    void greet(const char* who = "Derived") override { std::cout << "Derived::" << who; }
};
int main() { Derived d; Base* p = &d; p->greet(); }
```

- [ ] Prints Derived::Derived
- [x] Prints Derived::Base
- [ ] Prints Base::Base
- [ ] Compile error: an override cannot change a default argument

> Two mechanisms fire at different times: which function runs is decided at run time from the dynamic type (`Derived::greet`), but the default argument is substituted at compile time from the static type of the call expression (`Base*`, so `"Base"`). The mismatch `Derived::Base` is why style guides ban default arguments on virtual functions.

## quiz: Derived object passed by value as Base
tags: slicing, inheritance
track: core
difficulty: easy

print takes the shape by value.

```cpp
#include <iostream>
struct Shape  { virtual const char* name() const { return "Shape"; } };
struct Circle : Shape { const char* name() const override { return "Circle"; } };
void print(Shape s) { std::cout << s.name(); }
int main() { Circle c; print(c); }
```

- [ ] Prints Circle
- [x] Prints Shape
- [ ] Compile error: a Circle cannot initialize a Shape parameter
- [ ] Undefined behavior: the vtable pointer is corrupted

> Copy-initializing the `Shape` parameter from a `Circle` slices the object: only the `Shape` subobject is copied, and the parameter's dynamic type is exactly `Shape`. Virtual dispatch then works flawlessly — and correctly calls `Shape::name`. Polymorphism survives only through references and pointers; pass `const Shape&` instead.

## quiz: std::move on a const string
tags: move, const
track: core
difficulty: medium

The source string is const; the code tries to move from it.

```cpp
#include <iostream>
#include <string>
#include <utility>
int main() {
    const std::string src = "hello";
    std::string dst = std::move(src);
    std::cout << src;
}
```

- [ ] Prints nothing: src was moved out
- [ ] Compile error: a const object cannot be moved from
- [x] Prints hello: the "move" silently degrades to a copy
- [ ] Undefined behavior: reading a moved-from string

> `std::move(src)` yields `const std::string&&`, which cannot bind to the move constructor's `std::string&&` parameter (moving mutates the source). It binds happily to the copy constructor's `const std::string&`, so overload resolution silently picks the copy. It compiles, works, and leaves `src` untouched — a classic hidden pessimization worth a compiler-explorer check in hot paths.

## quiz: Flowing off the end of a non-void function
tags: ub, compilation
track: core
difficulty: medium

sign handles positive and negative — and forgets zero.

```cpp
int sign(int x) {
    if (x > 0) return 1;
    if (x < 0) return -1;
}
int main() { return sign(0); }
```

- [ ] Compile error: not all control paths return a value
- [ ] Compiles; sign(0) returns 0
- [x] Compiles (a warning at most); calling sign(0) is undefined behavior
- [ ] Compiles; sign(0) returns an unspecified but valid int

> Flowing off the end of a value-returning function is undefined behavior at the moment it happens — the standard does not require rejecting the code, so compilers only warn (`-Wreturn-type`). It is not merely "a garbage value": optimizers may assume the path is unreachable and delete the comparison logic around it. Only `main` is special, with an implicit `return 0;`.

## quiz: Incrementing unsigned max
tags: ub, integers
track: core
difficulty: medium

An unsigned counter sitting at its maximum value gets bumped once.

```cpp
#include <iostream>
#include <limits>
int main() {
    unsigned u = std::numeric_limits<unsigned>::max();
    ++u;
    std::cout << u;
}
```

- [ ] Undefined behavior: integer overflow
- [x] Prints 0: unsigned arithmetic wraps, guaranteed
- [ ] Prints -1
- [ ] Implementation-defined result

> Unsigned arithmetic is defined as modulo 2^N, so max + 1 is exactly 0 on every conforming implementation — this is not overflow at all in standard terms. It is *signed* overflow (`INT_MAX + 1`) that is undefined behavior. The asymmetry cuts both ways: unsigned loop indices never trip UB sanitizers, they just silently wrap — which is its own class of bug.

## quiz: push_back inside a range-for
tags: ub, iterators, containers
track: core
difficulty: medium

The loop appends to the very vector it is iterating.

```cpp
#include <vector>
int main() {
    std::vector<int> v{1, 2, 3};
    for (int x : v)
        if (x == 2) v.push_back(99);
}
```

- [ ] Compile error: the range is const inside a range-for
- [x] Compiles, but is undefined behavior
- [ ] Compiles; the loop also visits the appended 99
- [ ] Compiles and always terminates after exactly three iterations

> Range-for computes `begin()` and `end()` once, before the first iteration. `push_back` may reallocate, invalidating both iterators; even without reallocation the cached `end()` is stale. Incrementing or dereferencing an invalidated iterator is UB — and the fact that it usually "seems to work" on small vectors is exactly what makes it dangerous.

## quiz: auto grabs vector<bool>'s proxy
tags: auto, proxy-types, containers
track: core
difficulty: hard

A flag is "saved" into a local, then the vector is changed.

```cpp
#include <iostream>
#include <vector>
int main() {
    std::vector<bool> flags{true, true};
    auto first = flags[0];
    flags[0] = false;
    std::cout << first;
}
```

- [ ] Prints 1: first holds a copy of the bool
- [x] Prints 0: first is a proxy still referring into the vector
- [ ] Compile error: auto cannot deduce vector<bool>::reference
- [ ] Undefined behavior

> `std::vector<bool>` is bit-packed, so `operator[]` returns a proxy class — not `bool&` and not `bool`. `auto` faithfully deduces that proxy type, so `first` still refers to bit 0 and observes the later write, printing 0. Write `bool first = flags[0];` to force a real copy. Any proxy-returning API (expression templates, bitfields wrappers) breaks naive `auto` the same way.

## quiz: Derived overload hides the base one
tags: compilation, name-hiding, overloading
track: core
difficulty: medium

Base has show(int); Derived adds show(const char*); the call passes 42.

```cpp
#include <iostream>
struct Base    { void show(int) { std::cout << "int"; } };
struct Derived : Base { void show(const char*) { std::cout << "cstr"; } };
int main() {
    Derived d;
    d.show(42);
}
```

- [x] Compile error: Base::show is hidden and 42 won't convert to const char*
- [ ] Prints int: overload resolution merges both scopes
- [ ] Prints cstr: 42 converts to a null pointer
- [ ] Compile error: overloading across class scopes is ambiguous

> Name lookup stops at the first scope where the name is found: `Derived::show` hides *every* `Base::show`, so only `show(const char*)` is a candidate — and `42` (a non-literal-zero int expression aside, since C++11 no int converts implicitly to a pointer) has no viable conversion. Overload resolution never merges base and derived scopes. `using Base::show;` inside Derived un-hides the base overloads.

## quiz: Timer t(); — what did you just declare?
tags: compilation, parsing
track: core
difficulty: medium

The programmer "default-constructs" a timer with empty parentheses.

```cpp
#include <iostream>
struct Timer { int elapsed() const { return 0; } };
int main() {
    Timer t();
    std::cout << t.elapsed();
}
```

- [ ] Compiles and prints 0
- [x] Compile error: t is a function declaration, not an object
- [ ] Compiles; t is value-initialized
- [ ] Compile error: Timer has no user-provided default constructor

> This is the most vexing parse: anything that *can* be parsed as a declaration *is* one, so `Timer t();` declares a function `t` taking nothing and returning `Timer`. The error only surfaces at `t.elapsed()` — member access on a function is ill-formed. `Timer t;` or `Timer t{};` (braces can never declare a function) say what was meant.

## quiz: Copy-init through an explicit constructor
tags: compilation, conversions, explicit
track: core
difficulty: easy

Meters guards its constructor with explicit; the code initializes with =.

```cpp
struct Meters {
    explicit Meters(double v) : value(v) {}
    double value;
};
int main() {
    Meters m = 5.0;
    return static_cast<int>(m.value);
}
```

- [x] Compile error: copy-initialization cannot use an explicit constructor
- [ ] Compiles; = and direct-initialization are equivalent here
- [ ] Compiles; a temporary Meters is materialized then copied
- [ ] Compile error: explicit is not allowed on single-parameter constructors

> `Meters m = 5.0;` is copy-initialization, which considers only converting (non-explicit) constructors — an `explicit` constructor is simply not a candidate for the implicit `double → Meters` conversion. Direct-initialization, `Meters m(5.0);` or `Meters m{5.0};`, names the type deliberately and is allowed. That asymmetry is the entire point of `explicit`.

## quiz: Plain if where if constexpr was needed
tags: compilation, templates, constexpr
track: core
difficulty: hard

value_of dereferences pointers and passes other values through.

```cpp
#include <iostream>
#include <type_traits>
template <typename T>
auto value_of(T t) {
    if (std::is_pointer_v<T>) return *t;
    return t;
}
int main() { std::cout << value_of(42); }
```

- [ ] Compiles and prints 42: the pointer branch is never taken
- [x] Compile error: *t must compile for T = int even though the branch is dead
- [ ] Undefined behavior: an int is dereferenced
- [ ] Compile error: is_pointer_v cannot be used in a runtime condition

> A runtime `if` is just control flow: both branches are fully instantiated for every `T`, and for `T = int` the expression `*t` is ill-formed no matter how provably false the condition is. `if constexpr` is the fix — the untaken branch is discarded at instantiation time and never type-checked against this `T`. (The two `return`s with different types would also break `auto` deduction for pointer types — same cure.)

## quiz: auto with a multi-element braced initializer
tags: compilation, auto, initialization
track: core
difficulty: hard

Two autos, one brace pair each.

```cpp
int main() {
    auto a{1};
    auto b{2, 3};
    return a;
}
```

- [ ] Compiles: a is int, b is std::initializer_list<int>
- [x] Compile error: braced auto with more than one element is ill-formed
- [ ] Compiles: both are std::initializer_list<int>
- [ ] Compile error: auto can never be brace-initialized

> Since the C++17 rule change (N3922), direct-list-initialization of `auto` with exactly one element deduces the element's type — `a` is a plain `int` — and with more than one element it is ill-formed, full stop. Only the copy-list-init form `auto b = {2, 3};` still deduces `std::initializer_list<int>`. Older material predating the change gets this wrong.

## quiz: string_view of a temporary string
tags: ub, lifetime, string_view
track: core
difficulty: hard

A view is taken of a function's return value.

```cpp
#include <iostream>
#include <string>
#include <string_view>
std::string make_name() { return "temporary"; }
int main() {
    std::string_view sv = make_name();
    std::cout << sv;
}
```

- [ ] Compile error: string_view cannot bind to a temporary
- [ ] Prints temporary: the view extends the string's lifetime
- [x] Compiles, but printing sv is undefined behavior
- [ ] Prints an empty string, guaranteed

> The temporary `std::string` lives only until the end of the full-expression that initializes `sv`. Lifetime extension applies to temporaries bound *directly to references* — not to a class that squirrels away a pointer into the temporary's buffer, which is all `string_view` is. On the next line `sv` dangles and reading it is UB; it merely tends to "work" because the freed buffer is often still intact.
