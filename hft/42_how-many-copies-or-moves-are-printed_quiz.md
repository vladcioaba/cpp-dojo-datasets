## quiz: How many copies or moves are printed?
tags: move, copy-elision, rvo
track: hft

```cpp
struct S {
    S() {}
    S(const S&) { std::puts("copy"); }
    S(S&&)      { std::puts("move"); }
};
S make() { return S{}; }
int main() { S s = make(); }
```

- [x] 0 — guaranteed copy elision (C++17) constructs the object directly into `s`
- [ ] 1 move
- [ ] 1 copy
- [ ] 2 moves

> Since C++17, returning a prvalue whose type matches the return type, and initializing a variable from a prvalue, are *not* copies or moves — the object is materialized directly in the destination. No copy or move constructor is called (it need not even be accessible). So nothing prints. This is stronger than the pre-C++17 "as-if" RVO, which merely *permitted* elision.
