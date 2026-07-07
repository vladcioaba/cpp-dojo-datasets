## quiz: What decides whether a `vector` reallocation moves or copies its elements?
tags: move, noexcept, vector
track: hft

```cpp
struct Widget {
    std::vector<int> data;
    Widget(Widget&&) /* noexcept? */;
    Widget(const Widget&);
};
std::vector<Widget> v;   // grows past capacity -> reallocation
```

- [ ] It always moves in C++11 and later
- [x] It moves only if the move constructor is `noexcept` (or no copy ctor exists); otherwise it copies, to preserve the strong exception guarantee
- [ ] It always copies unless you call `reserve` first
- [ ] `noexcept` is irrelevant; only `-O2` enables the moves

> `vector` reallocation uses `move_if_noexcept`. If moving an element could throw and a copy constructor is available, it copies instead — because a throw partway through moving would leave both the old and new buffers in a broken state, violating the strong exception guarantee. So a non-`noexcept` move ctor silently degrades your `vector` growth to copies. Mark move constructors `noexcept`.
