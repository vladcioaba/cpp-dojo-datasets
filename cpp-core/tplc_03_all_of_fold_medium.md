## challenge: does the value pass every check?
tags: templates, variadic
track: core
difficulty: medium

Write a variadic function template `satisfiesAll(value, preds...)` taking one value and any number of callables. It returns `true` exactly when every `pred(value)` is true, implemented as a single fold expression over `&&`. Calling it with no predicates must return `true`, and evaluation must short-circuit: once one predicate says no, later predicates are never called.

hint: The pack holds the predicates, not the values — expand `preds(value)` and fold the results with one operator.
hint: `&&` is one of the operators whose unary fold is allowed over an empty pack — it folds to `true`, which gives you the no-predicates case for free.
hint: `return (... && preds(value));` — a unary left fold; because it expands to real `&&` operators, short-circuiting works exactly as in hand-written code.

```cpp
// starter
// True iff every pred(value) is true. No predicates -> true.
// One fold expression, short-circuiting preserved.
// TODO: write the variadic function template.
template <class T, class... Preds>
bool satisfiesAll(const T& value, Preds... preds) {
    return false; // TODO
}
```

```cpp
template <class T, class... Preds>
bool satisfiesAll(const T& value, Preds... preds) {
    return (... && preds(value));
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    auto positive = [](int x) { return x > 0; };
    auto even     = [](int x) { return x % 2 == 0; };
    auto small    = [](int x) { return x < 100; };
    assert(satisfiesAll(8, positive, even, small));
    assert(!satisfiesAll(7, positive, even));
    assert(!satisfiesAll(-2, positive, even, small));
    assert(satisfiesAll(42));                     // no predicates: vacuously true
    int calls = 0;
    auto reject = [&](int) { ++calls; return false; };
    auto record = [&](int) { ++calls; return true; };
    assert(!satisfiesAll(1, reject, record));
    assert(calls == 1);                           // fold short-circuited after the first false
    std::puts("PASS");
}
```

**Editorial:** The trick is realizing the fold runs over the *predicates* while `value` stays fixed: the pattern `preds(value)` is expanded per pack element, then stitched together with `&&`. For `satisfiesAll(x, p, q)` the fold `(... && preds(value))` expands to `p(x) && q(x)` — genuine built-in `&&`, so short-circuiting is preserved, which the call-counting assert proves. The empty pack is handled by the language itself: unary folds are ill-formed for an empty pack *except* for `&&` (folds to `true`), `||` (folds to `false`), and the comma operator — the identity elements of each operation. The same shape with `||` gives you `satisfiesAny` for free. Note the predicates are taken by value, the standard-library convention for callables — cheap for lambdas and function pointers.
