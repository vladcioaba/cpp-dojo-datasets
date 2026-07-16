## challenge: a sort wrapper that rejects the unsortable
tags: templates, concepts
track: core
difficulty: medium

`std::sort` on the wrong container fails with a wall of iterator errors from deep inside the algorithm. Fix the diagnostics at the boundary: define a concept `SortableRange` that holds when `std::begin(r)` yields a random-access iterator that satisfies `std::sortable`, and `std::end(r)` is valid. Then write `sortInPlace(r)` constrained on that concept, sorting the range with `std::sort`. Vectors, `std::array`, and raw arrays must pass; `std::list` and non-ranges must be rejected *by the constraint*.

hint: Constrain on evidence, not on names: a `requires` expression can demand that `std::begin(r)` compiles AND that its type models a standard iterator concept.
hint: A compound requirement checks an expression's type: `{ std::begin(r) } -> std::random_access_iterator;`. Add `requires std::sortable<decltype(std::begin(r))>;` for element movability/comparability — `std::sortable` alone is not enough, since a `std::list<int>` iterator satisfies it.
hint: `template <SortableRange R> void sortInPlace(R& r) { std::sort(std::begin(r), std::end(r)); }` — using `std::begin`/`std::end` (not members) is what lets raw arrays through.

```cpp
// starter
// concept SortableRange: std::begin(r) is a random-access, sortable iterator,
// std::end(r) valid. sortInPlace(r) sorts, and won't even match a std::list.
// TODO: define the concept and the constrained function template.
```

```cpp
template <class R>
concept SortableRange = requires(R& r) {
    { std::begin(r) } -> std::random_access_iterator;
    requires std::sortable<decltype(std::begin(r))>;
    std::end(r);
};

template <SortableRange R>
void sortInPlace(R& r) {
    std::sort(std::begin(r), std::end(r));
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> v{5, 3, 1, 4, 2};
    sortInPlace(v);
    assert((v == std::vector<int>{1, 2, 3, 4, 5}));
    std::array<double, 3> a{2.5, 0.5, 1.5};
    sortInPlace(a);
    assert(a[0] == 0.5 && a[2] == 2.5);
    int raw[4] = {9, 7, 8, 6};
    sortInPlace(raw);                      // std::begin works on raw arrays
    assert(raw[0] == 6 && raw[3] == 9);
    std::vector<std::string> w{"pear", "fig"};
    sortInPlace(w);
    assert(w[0] == "fig");
    static_assert(SortableRange<std::vector<int>>);
    static_assert(SortableRange<int[8]>);
    static_assert(!SortableRange<std::list<int>>);  // bidirectional only: rejected up front
    static_assert(!SortableRange<int>);             // no begin/end at all
    std::puts("PASS");
}
```

**Editorial:** The concept encodes `std::sort`'s real preconditions at the interface instead of letting them explode inside `<algorithm>`. Each requirement pulls weight: `{ std::begin(r) } -> std::random_access_iterator` demands both that the expression compiles and that its type models random access — which is what kills `std::list`, whose bidirectional iterators would otherwise slip through, since `std::sortable` (perhaps surprisingly) does not require random access, only that elements can be permuted and compared. The nested `requires std::sortable<...>` adds exactly that movability/comparability check, mirroring how `std::ranges::sort` itself is constrained (`random_access_iterator` + `sortable`). Using free `std::begin`/`std::end` rather than member calls is what admits raw C arrays. The payoff shows up in the error message: calling `sortInPlace(list)` now reports "constraints not satisfied: SortableRange" at the call site, one line, instead of a template backtrace from the sorting internals — and the `static_assert`s in the harness demonstrate the concept doubles as a queryable type trait.
