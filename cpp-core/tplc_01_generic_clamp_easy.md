## challenge: a clamp that works for any ordered type
tags: templates, deduction
track: core
difficulty: easy

Write a function template `clampValue` taking three parameters of type `const T&` — `value`, `low`, `high` — and returning `const T&`: `low` if `value < low`, `high` if `high < value`, otherwise `value` itself. Use only `operator<` for comparisons. Template argument deduction must make `clampValue(5, 1, 10)` work with no explicit template arguments.

hint: One type parameter is enough — all three arguments and the return value share the same type.
hint: Express both boundary checks with `operator<` alone: `value < low` for the lower bound, `high < value` for the upper.
hint: `template <class T> const T& clampValue(const T& value, const T& low, const T& high)` — return `low` when `value < low`, `high` when `high < value`, else `value`.

```cpp
// starter
// Return low if value < low, high if high < value, otherwise value.
// Deduction must allow clampValue(5, 1, 10) — no explicit <int> at the call.
// TODO: write the function template using only operator<.
```

```cpp
template <class T>
const T& clampValue(const T& value, const T& low, const T& high) {
    if (value < low) return low;
    if (high < value) return high;
    return value;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(clampValue(5, 1, 10) == 5);
    assert(clampValue(-3, 1, 10) == 1);
    assert(clampValue(42, 1, 10) == 10);
    assert(clampValue(2.5, 0.0, 1.0) == 1.0);
    std::string s = clampValue(std::string("m"), std::string("a"), std::string("z"));
    assert(s == "m");
    // returning const T& means no copy: in-range clamping hands back the object itself
    int v = 7;
    assert(&clampValue(v, 0, 100) == &v);
    std::puts("PASS");
}
```

**Editorial:** All three parameters mention the same `T`, so deduction requires the arguments to agree — `clampValue(5, 1, 10)` deduces `T = int` from every position, while a mixed call like `clampValue(5, 1.0, 10)` refuses to compile instead of silently converting, which is exactly the safety you want at a boundary check. Taking and returning `const T&` avoids copying potentially expensive types like `std::string`, and the address assert proves the in-range case returns the caller's own object. Using only `operator<` (never `>` or `<=`) mirrors `std::clamp` and the rest of the standard library, so any type that defines a single comparison works. One caution inherited from `std::clamp`: never call it with temporaries and bind the result to a reference that outlives the statement.
