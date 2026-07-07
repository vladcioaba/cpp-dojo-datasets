## fact: Type erasure — runtime polymorphism without inheritance
tags: type-erasure, value-semantics, polymorphism
track: design

**Type erasure** gives you virtual-style dispatch over *unrelated* types that share only a shape — none of them inherits from a common base. The wrapper holds an internal `Concept`/`Model` pair (the vtable is generated per stored type) behind a value-semantic handle. This is exactly how `std::function`, `std::any`, and `std::shared_ptr`'s deleter work.

The win: your `Circle` and `Square` need only a member `area()`; they stay decoupled, testable, and usable as plain values — no base class contaminating your domain types.

```cpp
class AnyShape {
    struct Concept { virtual double area() const = 0; virtual ~Concept() = default; };
    template <class T> struct Model : Concept {
        T obj; explicit Model(T o) : obj(std::move(o)) {}
        double area() const override { return obj.area(); }
    };
    std::unique_ptr<Concept> self_;
public:
    template <class T>
        requires (!std::same_as<std::remove_cvref_t<T>, AnyShape>)   // don't hijack copy/move
    AnyShape(T x) : self_(std::make_unique<Model<T>>(std::move(x))) {}
    double area() const { return self_->area(); }
};
```
