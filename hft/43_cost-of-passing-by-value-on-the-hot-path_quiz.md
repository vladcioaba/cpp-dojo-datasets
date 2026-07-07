## quiz: Cost of passing `std::shared_ptr` by value on the hot path
tags: smart-pointers, atomics, hot-path
track: hft

```cpp
void process(std::shared_ptr<Order> o);   // called millions of times per second
```

- [ ] Free — copying a `shared_ptr` is just copying a pointer
- [x] Each copy is an atomic increment of the refcount and each destruction an atomic decrement — synchronized RMW operations that are expensive under contention; pass by `const&` or by raw reference on the hot path
- [ ] The copy deep-copies the pointed-to `Order`
- [ ] Copies are cheap but the destructor allocates

> A `shared_ptr` copy bumps the control block's reference count with an atomic `fetch_add`, and the destructor does an atomic `fetch_sub` (with a release/acquire fence so the last one frees safely). Atomic RMWs are far costlier than a plain pointer copy, especially when multiple cores touch the same control block. When you are not transferring ownership, pass `const std::shared_ptr&`, or better a raw `Order*`/`Order&`.
