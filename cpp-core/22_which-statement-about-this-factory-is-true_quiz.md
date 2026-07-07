## quiz: Which statement about this factory is true?
tags: patterns, factory

```cpp
std::unique_ptr<Shape> make_shape(std::string_view kind) {
    if (kind == "circle") return std::make_unique<Circle>();
    if (kind == "square") return std::make_unique<Square>();
    return nullptr;
}
```

- [x] Callers own the result and never see concrete types — classic Factory decoupling
- [ ] It leaks unless callers call delete
- [ ] It should return shared_ptr, unique_ptr can't hold derived types
- [ ] Returning nullptr from a unique_ptr function is undefined behavior

> The factory function centralizes construction, returns ownership via `unique_ptr<Base>` (upcast from `unique_ptr<Derived>` works out of the box — but remember `Shape` needs a virtual destructor). A null `unique_ptr` is a perfectly valid "not found" result, though `std::optional`-style designs or exceptions are alternatives.
