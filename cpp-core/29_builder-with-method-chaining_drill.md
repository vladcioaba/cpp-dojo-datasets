## exercise: Builder with method chaining
tags: patterns, builder

Write the member function `title` for `QueryBuilder`: it takes `std::string t`, assigns it with `title_ = std::move(t);`, and returns `*this` by reference so calls chain.

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
