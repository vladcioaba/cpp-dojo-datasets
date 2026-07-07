## fact: string_view is a borrow, not a string
tags: core, lifetime

`std::string_view` is a pointer + length into memory it does not own. It is perfect for read-only parameters — no allocation, accepts literals and `std::string` alike. It is a landmine as a return value or a member: the moment the underlying string dies, the view dangles.

```cpp
std::string_view bad() {
    std::string s = "temporary";
    return s;              // dangling view — UB on use
}
```
