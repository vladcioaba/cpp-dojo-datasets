## quiz: What happens here?
tags: core, smart-pointers

```cpp
std::unique_ptr<int> a = std::make_unique<int>(42);
std::unique_ptr<int> b = a;
```

- [x] Compile error — unique_ptr's copy constructor is deleted
- [ ] b becomes a dangling pointer
- [ ] Both point to 42; last one to die frees it
- [ ] Runtime crash

> Unique ownership means no copies: the copy constructor and copy assignment are `= delete`. Transfer requires an explicit `std::unique_ptr<int> b = std::move(a);`, after which `a` is null.
