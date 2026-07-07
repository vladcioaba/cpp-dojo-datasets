## fact: Strategy pattern is often just std::function
tags: patterns, strategy

The GoF Strategy pattern — swap an algorithm at runtime — needed an interface, virtual method, and concrete class per strategy. Modern C++ collapses it to a `std::function` member (or a template parameter if the strategy is fixed at compile time).

```cpp
class Sorter {
    std::function<bool(int,int)> cmp = std::less<int>{};
public:
    void set_strategy(std::function<bool(int,int)> f) { cmp = std::move(f); }
};
```

Virtual hierarchy still wins when strategies carry heavy state or need multiple methods.
