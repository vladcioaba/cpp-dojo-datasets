## challenge: get<I> for a hand-rolled tuple
tags: templates, variadic, if-constexpr
track: core
difficulty: hard

The starter defines a minimal recursive tuple: `Tuple<Head, Tail...>` stores `head` plus a nested `Tuple<Tail...>`. Write the missing accessor: a function template `get<I>(t)` that returns a *reference* to the I-th element, so callers can both read and assign through it. Index the elements at compile time — either with `if constexpr` recursion or with recursive overloads.

hint: The element's type changes with `I`, so the return type must be deduced — `auto&` — and the index must be a non-type template parameter, not a function argument.
hint: Base case: `I == 0` returns `t.head`. Otherwise recurse into `t.tail` with `I - 1`. A runtime `if` cannot do this — both branches would need to compile with the same return type.
hint: `template <std::size_t I, class Head, class... Tail> constexpr auto& get(Tuple<Head, Tail...>& t) { if constexpr (I == 0) return t.head; else return get<I - 1>(t.tail); }`

```cpp
// starter
template <class... Ts> struct Tuple;

template <> struct Tuple<> {};

template <class Head, class... Tail>
struct Tuple<Head, Tail...> {
    Head head;
    Tuple<Tail...> tail;
    constexpr Tuple(Head h, Tail... t) : head(h), tail(t...) {}
};

// TODO: write get<I>(t) returning a reference to the I-th element.
// Reads AND writes must work: get<0>(t) = 7;
```

```cpp
template <class... Ts> struct Tuple;

template <> struct Tuple<> {};

template <class Head, class... Tail>
struct Tuple<Head, Tail...> {
    Head head;
    Tuple<Tail...> tail;
    constexpr Tuple(Head h, Tail... t) : head(h), tail(t...) {}
};

template <std::size_t I, class Head, class... Tail>
constexpr auto& get(Tuple<Head, Tail...>& t) {
    if constexpr (I == 0) {
        return t.head;
    } else {
        return get<I - 1>(t.tail);
    }
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Tuple<int, std::string, double> t(42, std::string("mid"), 2.5);
    assert(get<0>(t) == 42);
    assert(get<1>(t) == "mid");
    assert(get<2>(t) == 2.5);
    get<0>(t) = 7;                 // must return a real reference
    get<1>(t) += "dle";
    assert(get<0>(t) == 7);
    assert(get<1>(t) == "middle");
    static_assert(std::is_same_v<decltype(get<2>(t)), double&>);
    static_assert(std::is_same_v<decltype(get<1>(t)), std::string&>);
    std::puts("PASS");
}
```

**Editorial:** This is `std::tuple`'s core idea in miniature: storage by recursive nesting, access by compile-time index arithmetic. Two details carry the exercise. First, the index must be a template parameter because each `I` selects a *different type* — a runtime parameter could never do that. Second, `if constexpr` is essential: with a plain `if`, both branches are compiled for every instantiation, and at `I == 0` the else-branch would instantiate `get<-1>` (which underflows `std::size_t` and recurses forever), while the two branches would also disagree on return type. `if constexpr` discards the untaken branch entirely, so `auto&` deduces from exactly one `return`. An out-of-range index fails loudly — `get<3>` eventually tries to recurse into `Tuple<>`, which no overload matches. The real `std::tuple` adds const/rvalue overloads and usually flattens storage via multiple inheritance for compile-time speed, but the indexing principle is the one you just wrote.
