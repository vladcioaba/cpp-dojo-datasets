## challenge: convert to any container — template template parameters
tags: templates, template-template
track: core
difficulty: hard

Write `convertTo<Target>(src)` — a function template that copies a `const std::vector<T>&` into a *different container template* chosen by the caller: `convertTo<std::deque>(v)` yields `std::deque<int>`, `convertTo<std::list>(v)` yields `std::list<int>`, `convertTo<std::set>(v)` yields `std::set<int>`. The caller names the template itself, not a full type — so the parameter must be a template template parameter, and your function applies it to `T`. Construct the result from the iterator range `src.begin(), src.end()`.

hint: A normal type parameter can hold `std::deque<int>`, but not `std::deque` itself. Declaring "a parameter that is a template" looks like: `template <template <class...> class Target, ...>`.
hint: Declare the pack form `template <class...> class Target`, not `template <class> class Target`: `std::deque`, `std::list`, and `std::set` all carry extra defaulted parameters (allocator, comparator), and since C++17 the variadic form matches them all.
hint: `template <template <class...> class Target, class T> Target<T> convertTo(const std::vector<T>& src) { return Target<T>(src.begin(), src.end()); }` — `T` is still deduced from `src`; only `Target` is explicit at the call.

```cpp
// starter
// convertTo<std::deque>(vec)  -> std::deque<T> with the same elements
// convertTo<std::list>(vec)   -> std::list<T>
// convertTo<std::set>(vec)    -> std::set<T> (duplicates collapse)
// TODO: write the function template taking a template template parameter.
```

```cpp
template <template <class...> class Target, class T>
Target<T> convertTo(const std::vector<T>& src) {
    return Target<T>(src.begin(), src.end());
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> v{1, 2, 3};
    auto d = convertTo<std::deque>(v);
    static_assert(std::is_same_v<decltype(d), std::deque<int>>);
    assert(d.size() == 3 && d.front() == 1 && d.back() == 3);
    auto l = convertTo<std::list>(v);
    static_assert(std::is_same_v<decltype(l), std::list<int>>);
    assert(std::equal(v.begin(), v.end(), l.begin()));
    std::vector<std::string> words{"beta", "alpha"};
    auto ws = convertTo<std::set>(words);              // element type follows the source
    static_assert(std::is_same_v<decltype(ws), std::set<std::string>>);
    assert(*ws.begin() == "alpha");
    auto s = convertTo<std::set>(std::vector<int>{3, 1, 3, 2});
    assert((s == std::set<int>{1, 2, 3}));             // duplicates collapsed
    std::puts("PASS");
}
```

**Editorial:** A template template parameter abstracts over the container *family* rather than a finished type: the caller supplies `std::deque` — a recipe still waiting for arguments — and the function applies it to the element type it deduced from the source. Neither a type parameter nor a value parameter can express that. The declaration syntax is the hurdle: `template <template <class...> class Target, class T>` reads inside-out as "Target is itself a template taking any number of type parameters." The pack matters — `std::deque` is really `template <class T, class Allocator>` and `std::set` drags a comparator too, so a strict `template <class> class` parameter wouldn't match them (C++17 relaxed matching lets the variadic form accept templates with defaulted extras). Note the mixed deduction at the call site: `Target` must be explicit because nothing in the arguments mentions it, while `T` is still deduced from `src` — explicit-then-deduced ordering is a common pattern. In modern code CTAD or an `auto`-returning lambda sometimes replaces this technique, but rebinding a container family — as allocators and `std::pmr` do internally — still runs on template template parameters.
