## challenge: teach Pair that a literal means std::string
tags: templates, ctad
track: core
difficulty: easy

The starter defines an aggregate `Pair<A, B>`. Thanks to C++20 aggregate CTAD, `Pair p{1, 2.5}` already deduces `Pair<int, double>` — but `Pair p{"port", 8080}` deduces `A = const char*`, which is almost never what you want. Write deduction guides so string literals deduce as `std::string` in either position: `Pair{"port", 8080}` → `Pair<std::string, int>`, `Pair{1, "one"}` → `Pair<int, std::string>`, `Pair{"k", "v"}` → `Pair<std::string, std::string>`. Plain cases like `Pair{1, 2.5}` must keep working — and there is a trap there: the implicit aggregate deduction candidate exists only while you have written *no* guides of your own.

hint: A deduction guide looks like a constructor declaration with a trailing arrow: `Pair(X, Y) -> Pair<...>;` at namespace scope. It only steers deduction — it adds no constructor. And the moment you declare one, implicit aggregate CTAD is gone: restore the plain case yourself with `template <class A, class B> Pair(A, B) -> Pair<A, B>;`.
hint: You need a guide per literal position: first argument `const char*`, second argument `const char*`. Keep the other position generic — partial ordering prefers these over the generic `(A, B)` guide when a literal is present.
hint: `Pair{"k", "v"}` matches both single-position guides ambiguously — add a fourth, non-template guide `Pair(const char*, const char*) -> Pair<std::string, std::string>;` which wins because non-templates beat templates.

```cpp
// starter
template <class A, class B>
struct Pair {
    A first;
    B second;
};

// TODO: add deduction guides so that string literals deduce as std::string:
//   Pair{"port", 8080}  ->  Pair<std::string, int>
//   Pair{1, "one"}      ->  Pair<int, std::string>
//   Pair{"k", "v"}      ->  Pair<std::string, std::string>
```

```cpp
template <class A, class B>
struct Pair {
    A first;
    B second;
};

template <class A, class B> Pair(A, B) -> Pair<A, B>;  // restore the plain case
template <class B> Pair(const char*, B) -> Pair<std::string, B>;
template <class A> Pair(A, const char*) -> Pair<A, std::string>;
Pair(const char*, const char*) -> Pair<std::string, std::string>;
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Pair p1{"port", 8080};
    static_assert(std::is_same_v<decltype(p1), Pair<std::string, int>>);
    Pair p2{1, "one"};
    static_assert(std::is_same_v<decltype(p2), Pair<int, std::string>>);
    Pair p3{"k", "v"};
    static_assert(std::is_same_v<decltype(p3), Pair<std::string, std::string>>);
    Pair p4{1, 2.5};                       // aggregate CTAD still does the plain cases
    static_assert(std::is_same_v<decltype(p4), Pair<int, double>>);
    assert(p1.first.size() == 4 && p1.second == 8080);
    assert(p2.second == "one");
    assert(p3.first + p3.second == "kv");
    std::puts("PASS");
}
```

**Editorial:** Without guides, deduction takes the arguments literally: a string literal is a `const char[5]` that decays to `const char*`, so `Pair{"port", 8080}` would give you a pair holding a raw pointer — comparisons become pointer comparisons and lifetime bugs follow. A deduction guide rewrites the conclusion without touching the class: `template <class B> Pair(const char*, B) -> Pair<std::string, B>` says "when the first argument is a `const char*`, deduce `std::string` instead." The big trap is the generic guide: the standard adds the implicit aggregate deduction candidate only when *no* user guides exist, so declaring your first guide silently breaks `Pair{1, 2.5}` until you restore `Pair(A, B) -> Pair<A, B>` yourself (g++ diagnoses this as "deduction failed" at the innocent-looking call site). Overload resolution then sorts the rest: partial ordering prefers `(const char*, B)` over `(A, B)` when a literal is present, and the two-literal call — ambiguous between the two single-position guides — goes to the non-template fourth guide, since non-templates beat templates. Initialization stays plain aggregate init; the guides only pick `A` and `B`, and `const char*` converts to `std::string` member-wise.
