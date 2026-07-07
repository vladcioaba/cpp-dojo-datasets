## exercise: Observer — subscribe and notify
tags: patterns, observer

`Button` stores callbacks in `std::vector<std::function<void()>> handlers`. Write two member functions: `on_click` taking `std::function<void()> h` and appending it with `push_back(std::move(h))`, and `click()` calling every handler `h` in `handlers` with a range-for over `auto& h`.

hint: Store the subscribers as callables, then iterate over them and invoke each when the event fires.
hint: `push_back(std::move(h))` to register a handler; a range-for calling `h()` to notify them all.

```cpp
// starter
class Button {
    std::vector<std::function<void()>> handlers;
public:
    // your code here
};
```

```cpp
void on_click(std::function<void()> h) {
    handlers.push_back(std::move(h));
}
void click() {
    for (auto& h : handlers) h();
}
```

```cpp
// harness
#include <vector>
#include <functional>
#include <cstdio>
class Button {
    std::vector<std::function<void()>> handlers;
public:
//__USER__
};
int main() {
    Button b;
    int n = 0;
    b.on_click([&] { ++n; });
    b.on_click([&] { n += 10; });
    b.click();
    if (n != 11) return 1;
    std::puts("PASS");
}
```

**Editorial:** The observer pattern keeps a list of `std::function<void()>` callbacks: `on_click` registers one (moving it in to avoid a copy) and `click` fires the event by invoking each in turn. Using `std::function` decouples the subject from concrete observer types. The drill teaches callback registration and notification.
