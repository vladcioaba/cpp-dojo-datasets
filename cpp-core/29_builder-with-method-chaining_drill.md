## exercise: Builder with method chaining
tags: patterns, builder

Write the member function `title` for `QueryBuilder`: it takes `std::string t`, assigns it with `title_ = std::move(t);`, and returns `*this` by reference so calls chain.

hint: To let calls chain, each setter must hand back the very object it was called on.
hint: Return `*this` by reference from the member function.

```cpp
// starter
class QueryBuilder {
    std::string title_;
public:
    // your code here
};
```

```cpp
QueryBuilder& title(std::string t) {
    title_ = std::move(t);
    return *this;
}
```

```cpp
// harness
#include <string>
#include <cstdio>
class QueryBuilder {
    std::string title_;
public:
//__USER__
    const std::string& get() const { return title_; }
};
int main() {
    QueryBuilder q;
    if (&q.title("a") != &q) return 1;
    q.title("x").title("y");
    if (q.get() != "y") return 1;
    std::puts("PASS");
}
```

**Editorial:** A builder's setters return `*this` by reference so calls chain fluently (`q.title(...).body(...)`); moving the string argument in avoids an extra copy. The pattern assembles a complex object step by step. The drill teaches method chaining via returning a self-reference.
