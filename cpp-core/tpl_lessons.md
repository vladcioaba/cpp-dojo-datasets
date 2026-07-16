## fact: Function templates — write it once, deduce the rest
tags: templates, deduction
track: core

A function template is a recipe the compiler stamps into a real function the moment you call it. Write the algorithm once with the type as a parameter, and let template argument deduction figure out `T` from the call site: `maxof(2, 3)` instantiates `maxof<int>`, and the same source line handles strings, doubles, or your own types. Deduction works by matching the parameter's shape against the argument's type — a `const T&` parameter given an `int` argument deduces `T = int`.

Two rules trip people up. First, deduction never performs conversions: `maxof(1, 2.5)` fails to compile because `T` cannot be both `int` and `double`; you resolve it by casting one argument or by writing `maxof<double>(1, 2.5)` — explicit template arguments switch deduction off entirely. Second, by-value parameters decay their arguments: top-level `const`, references, and array-ness are stripped, which is why `template <class T> void f(T x)` always receives a plain copy.

Since C++20 you can also write `auto` parameters — an abbreviated function template that behaves identically with less ceremony. Prefer letting deduction work: call sites stay clean, and changing an argument's type doesn't ripple through every caller.

```cpp
template <class T>
const T& maxof(const T& a, const T& b) { return a < b ? b : a; }

int  i = maxof(2, 3);                 // T deduced as int
auto s = maxof(std::string("a"), std::string("b"));
// maxof(1, 2.5);                     // error: T can't be int AND double
auto d = maxof<double>(1, 2.5);       // explicit argument: deduction off

void print(const auto& x);            // C++20 abbreviated function template
```

## fact: Class templates and CTAD — constructors that deduce
tags: templates, ctad
track: core

Class templates parameterize a type rather than a function: `std::vector<T>`, `std::pair<A, B>`, your own `Matrix<T>`. Before C++17 you always spelled the arguments out (`std::pair<int, std::string> p{1, "hi"}`) or reached for a maker function like `std::make_pair`, whose whole reason to exist was that constructors couldn't drive deduction.

Class template argument deduction (CTAD) fixed that: `std::pair p{1, 2.5}` deduces `pair<int, double>` straight from the constructor arguments, and `std::vector v{1, 2, 3}` deduces `vector<int>`. Under the hood the compiler builds a set of hypothetical function templates from the constructors — plus any deduction guides — and runs ordinary overload resolution on them.

Deduction guides matter when constructor parameter types shouldn't be taken literally. The standard library ships one so constructing a vector from two iterators deduces the element type, not the iterator type. You can write your own — for example, steering string literals to `std::string` instead of letting them decay to `const char*`. Since C++20, aggregates get implicit CTAD too, no constructor required — with one trap: the implicit aggregate candidate exists only while you have declared *no* guides of your own, so your first guide must be accompanied by a generic `Pair(A, B) -> Pair<A, B>` restoring the plain case. When deduction surprises you, spell the arguments explicitly: CTAD is a convenience, not an obligation.

```cpp
template <class A, class B>
struct Pair { A first; B second; };

template <class A, class B> Pair(A, B) -> Pair<A, B>;   // restore the plain case
template <class B> Pair(const char*, B) -> Pair<std::string, B>;

Pair p1{1, 2.5};         // Pair<int, double> via the generic guide
Pair p2{"port", 8080};   // Pair<std::string, int>: more specialized guide wins
std::vector v{1, 2, 3};  // std::vector<int>
```

## fact: Specialization — carving exceptions out of a template
tags: templates, specialization
track: core

A template is a general recipe; specialization is how you carve out exceptions. A full specialization replaces the implementation for one exact argument set: `template <> struct Hash<std::string> {...};` says "for `std::string`, use this instead." A partial specialization keeps some parameters open while constraining the shape: `template <class T> struct Traits<T*>` matches every pointer, `template <class T> struct Traits<std::vector<T>>` matches every vector. The compiler always picks the most specialized match, so the primary template becomes the fallback.

Two caveats. Only class templates and variable templates support partial specialization — function templates can only be fully specialized, and full specializations don't participate in overload resolution the way you'd expect, so the standing advice is: overload functions, specialize classes. Second, a specialization must be visible before the first use that would instantiate the primary template, or you get ODR trouble that no compiler is required to diagnose.

When is specialization the right tool? Type-keyed customization: traits classes (`std::iterator_traits`), `std::hash` for your own types, `std::tuple_size` to enable structured bindings. When you merely want different behavior per category of type inside one function, prefer `if constexpr` or concept-constrained overloads — reserve specialization for customizing types, not steering logic.

```cpp
template <class T> struct Traits {          // primary: the fallback
    static constexpr const char* name = "value";
};
template <class T> struct Traits<T*> {      // partial: any pointer
    static constexpr const char* name = "pointer";
};
template <> struct Traits<bool> {           // full: exactly bool
    static constexpr const char* name = "flag";
};

static_assert(std::string_view(Traits<int*>::name) == "pointer");
static_assert(std::string_view(Traits<bool>::name) == "flag");
```

## fact: Variadic templates — fold, don't recurse
tags: templates, variadic
track: core

Variadic templates let one template accept any number of arguments: `template <class... Ts>` declares a template parameter pack, `Ts... args` the matching function parameter pack, and `sizeof...(Ts)` counts it. Before C++17, consuming a pack meant peel-and-recurse: handle the first argument, forward the rest to a recursive call, and write a separate base-case overload for the empty pack.

Fold expressions replaced most of that ceremony. `(args + ...)` is a unary fold — it expands the pack with `+` between the elements — and `(args + ... + 0)` is a binary fold that supplies an initial value, which also rescues the empty pack (an empty unary `+` fold is ill-formed, while `&&` folds to `true` and `||` to `false` by definition). Folds work with any binary operator, and the comma operator turns a fold into a loop: `((std::cout << args << ' '), ...)` prints every argument in order.

The pack expansion pattern `f(args)...` transforms elementwise before folding, so `(... && pred(args))` checks a predicate over the whole pack with short-circuiting intact. Reach for recursion only when you need index-by-index control; a fold says "combine everything with this operator" in one readable line.

```cpp
template <class... Ts>
auto sum(Ts... args) { return (args + ... + 0); }   // binary fold: empty-safe

template <class... Ts>
void printAll(const Ts&... args) {
    ((std::cout << args << ' '), ...);              // comma fold as a loop
}

template <class P, class... Ts>
bool anyMatch(P pred, const Ts&... vals) {
    return (... || pred(vals));                     // expand, then fold
}
```

## fact: From SFINAE to concepts — constraints you can read
tags: templates, concepts
track: core

SFINAE — "substitution failure is not an error" — is the old mechanism for switching templates on and off: if substituting deduced types into a signature produces an invalid type, that candidate is silently dropped from overload resolution instead of failing the build. A decade of C++ ran on `std::enable_if` exploiting this. It worked, but `template <class T, std::enable_if_t<std::is_integral_v<T>, int> = 0>` reads like an incantation, and a failed match produces error novels instead of an explanation.

C++20 concepts express the same idea directly. A concept is a named, reusable compile-time predicate: `template <class T> concept Number = std::integral<T> || std::floating_point<T>;`. You apply it as a constrained parameter (`template <Number T>`), in a requires-clause (`template <class T> requires Number<T>`), or via a `requires` expression that checks syntax — `requires(T t) { t.begin(); }` holds exactly when the expressions inside would compile.

The wins are concrete: constraints sit in the signature where readers look; error messages name the requirement that failed instead of dumping a substitution trace; and overloads are ordered by which constraint subsumes which, replacing `enable_if` mutual-exclusion gymnastics. In new C++20 code, treat `enable_if` as legacy — anything it can do, a requires-clause states more honestly.

```cpp
// The old way: enable_if gymnastics
template <class T, std::enable_if_t<std::is_integral_v<T>, int> = 0>
T twiceOld(T x) { return x * 2; }

// C++20: say what you mean
template <class T>
concept HasSize = requires(const T& t) {
    { t.size() } -> std::convertible_to<std::size_t>;
};

std::size_t sizeOf(const HasSize auto& c) { return c.size(); }

template <class T> requires std::integral<T>
T twice(T x) { return x * 2; }
```

## fact: Non-type template parameters — values baked into types
tags: templates, nttp
track: core

Template parameters don't have to be types. A non-type template parameter (NTTP) bakes a value into the type itself: `std::array<int, 4>` carries its size at compile time, which is why the compiler can stack-allocate it, unroll loops over it — and why `std::array<int, 3>` is simply a different type that never mixes with it. Classic NTTPs are integers, enums, pointers, and references: `template <std::size_t N> struct Ring { int data[N]; };` — each distinct value of `N` produces a distinct instantiation with its own statics.

Values also drive compile-time computation. A `constexpr` function template with an `unsigned N` parameter computes during compilation, and the result is a constant expression usable in `static_assert` or as another template argument.

C++20 widened what a value can be: floating-point NTTPs are now legal, and so are class-type NTTPs, provided the type is "structural" — public members, all of them structural themselves, no custom `operator==`. That unlocks passing small configuration structs and fixed-capacity strings as template arguments, which older codebases faked with integer packs. Use NTTPs for values that genuinely shape the type — sizes, alignments, policies, units. Anything that varies at run time belongs in a constructor argument instead.

```cpp
template <std::size_t N>
struct Buffer { std::array<std::byte, N> data{}; };  // size is part of the type

template <unsigned N>
constexpr unsigned long long factorial() {
    if constexpr (N == 0) return 1;
    else return N * factorial<N - 1>();
}
static_assert(factorial<10>() == 3628800);

struct Limits { int lo; int hi; };        // structural: OK as C++20 NTTP
template <Limits L> int clampTo(int v) {
    return v < L.lo ? L.lo : (v > L.hi ? L.hi : v);
}
int capped = clampTo<Limits{0, 100}>(250);  // 100
```

## fact: Dependent names — why templates demand typename and template
tags: templates, dependent-names
track: core

Inside a template, names that depend on a template parameter are "dependent", and the compiler parses the template in two phases. Phase one, at definition time, checks everything non-dependent — syntax errors and unknown non-dependent names are diagnosed even if you never instantiate the template. Phase two, at instantiation, resolves the dependent names against the actual template arguments.

The catch: during phase one the compiler cannot know what a dependent name refers to, so it assumes the worst. In `T::iterator it;` the name `T::iterator` could be a type or a static data member, and the parser defaults to "not a type." You must promise it's a type: `typename T::iterator it;`. The same ambiguity hits member templates: in `t.template get<0>()`, without the `template` keyword the `<` parses as less-than and the expression falls apart.

Two-phase lookup also explains a classic inheritance surprise: members of a dependent base class are invisible to unqualified lookup in phase one, so inside `struct Derived : Base<T>` you must write `this->helper()` or `Base<T>::helper()` to defer the lookup to phase two. The rules feel bureaucratic, but they let templates be checked as early as possible — and g++ and clang both enforce them strictly.

```cpp
template <class T>
struct Base { void helper() {} };

template <class T>
struct Derived : Base<T> {
    typename T::value_type front(const T& c) {  // typename: dependent type
        return *c.begin();
    }
    template <class U>
    void relay(U& u) {
        u.template process<0>();                // template: dependent member template
    }
    void run() { this->helper(); }              // dependent base: this-> required
};
```

## fact: CRTP — static polymorphism, and when virtual still wins
tags: templates, crtp
track: core

The Curiously Recurring Template Pattern makes a base class know its derived type at compile time: `struct Widget : Counted<Widget>`. Inside `Counted<D>`, `static_cast<D*>(this)` reaches the derived object with zero runtime machinery — no vtable pointer in the object, no indirect call, full inlining. This is static polymorphism: the set of "overrides" is fixed when the program compiles.

Compare virtual dispatch. A virtual call costs a vtable indirection and usually blocks inlining, but it lets you store heterogeneous objects behind one `Base*` and pick behavior at run time. CRTP inverts the trade: `Counted<Widget>` and `Counted<Gadget>` are different base types, so there is no common pointer type and no runtime substitution — but every call compiles down to a direct, inlinable call, and each instantiation gets its own static members. That last property is exactly what makes per-derived-type counters and registries work.

Use CRTP for mixins (`std::enable_shared_from_this`, comparison-operator generators, instance counting) and zero-overhead interfaces on hot paths, where the concrete type is always known at the call site. Use `virtual` when callers genuinely hold mixed types in one container. C++23's "deducing this" expresses many CRTP mixins with less ceremony, but the pattern remains everywhere in production code.

```cpp
template <class Derived>
struct Counted {
    inline static int alive = 0;
    Counted() { ++alive; }
    ~Counted() { --alive; }
};

struct Widget : Counted<Widget> {};
struct Gadget : Counted<Gadget> {};  // its own counter: distinct base type

// virtual: one Base*, runtime choice, vtable indirection
// CRTP:   distinct bases, compile-time choice, direct inlined calls
```
