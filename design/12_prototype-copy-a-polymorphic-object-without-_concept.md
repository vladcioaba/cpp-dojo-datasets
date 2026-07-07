## fact: Prototype — copy a polymorphic object without knowing its type
tags: patterns, creational, prototype, clone
track: design

You hold a `Shape*` and need a duplicate, but you don't know if it's a `Circle` or a `Square`. A copy constructor won't help — it slices, copying only the base subobject. The **Prototype** pattern adds a virtual `clone()`: each concrete type copies *itself* and returns a fresh owning pointer.

This is the correct way to "polymorphic copy" in C++, and the foundation for value-semantic wrappers over polymorphic hierarchies.

```cpp
struct Shape {
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual ~Shape() = default;
};
struct Circle : Shape {
    double r;
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);   // copies the concrete Circle, no slicing
    }
};
```
