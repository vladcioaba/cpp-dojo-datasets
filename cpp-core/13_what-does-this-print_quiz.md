## quiz: What does this print?
tags: core, virtual

```cpp
struct Base {
    Base() { who(); }
    virtual void who() { std::cout << "Base "; }
};
struct Derived : Base {
    void who() override { std::cout << "Derived "; }
};
int main() { Derived d; d.who(); }
```

- [ ] Derived Derived
- [x] Base Derived
- [ ] Derived Base
- [ ] Undefined behavior

> Inside a constructor, the object's dynamic type is the class being constructed. `Base()` runs before `Derived` exists, so the call inside it dispatches to `Base::who`. Virtual dispatch "starts working" per-level as construction proceeds.
