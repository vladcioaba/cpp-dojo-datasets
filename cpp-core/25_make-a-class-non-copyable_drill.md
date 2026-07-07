## exercise: Make a class non-copyable
tags: raii, core

`Connection` must never be copied. Delete its copy constructor and copy assignment operator (in that order).

hint: You want the compiler to reject any attempt to copy — state that explicitly rather than hiding the functions.
hint: Mark the copy constructor and the copy assignment operator `= delete`.

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

**Editorial:** Marking the copy constructor and copy assignment operator `= delete` turns any copy attempt into a compile-time error, which is clearer and safer than the old private-and-undefined trick. It is the standard idiom for move-only or unique-ownership types. The drill teaches explicitly deleting special member functions.
