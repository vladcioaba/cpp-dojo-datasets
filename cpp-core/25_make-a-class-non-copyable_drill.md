## exercise: Make a class non-copyable
tags: raii, core

`Connection` must never be copied. Delete its copy constructor and copy assignment operator (in that order).

```cpp
// starter
class Connection {
public:
    // your two lines here
};
```

```cpp
Connection(const Connection&) = delete;
Connection& operator=(const Connection&) = delete;
```

```cpp
// harness
#include <cstdio>
#include <type_traits>
class Connection {
public:
//__USER__
};
static_assert(!std::is_copy_constructible_v<Connection>);
static_assert(!std::is_copy_assignable_v<Connection>);
int main() { std::puts("PASS"); }
```
