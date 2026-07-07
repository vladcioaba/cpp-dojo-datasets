## quiz: How many copy constructions happen?
tags: move, core

```cpp
std::vector<std::string> v;
std::string s = "hello";
v.push_back(std::move(s));
v.push_back("world");
```

- [x] 0
- [ ] 1
- [ ] 2
- [ ] Depends on the compiler

> `std::move(s)` makes the first push_back call the move overload. `"world"` constructs a temporary `std::string` which is an rvalue — moved too. (A reallocation between the two calls would move, not copy, since `std::string`'s move is `noexcept`.)
