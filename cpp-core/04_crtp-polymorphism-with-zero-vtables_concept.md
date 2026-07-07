## fact: CRTP — polymorphism with zero vtables
tags: patterns, crtp, templates

The Curiously Recurring Template Pattern: a base class templated on its own derived class. The base can call derived methods via `static_cast<Derived*>(this)` — resolved at compile time, inlined, no virtual dispatch cost.

```cpp
template <class Derived>
struct Shape {
    double area() const {
        return static_cast<const Derived*>(this)->area_impl();
    }
};
struct Circle : Shape<Circle> {
    double area_impl() const { return 3.14159 * r * r; }
    double r = 1.0;
};
```

Used all over the standard library and libraries like Eigen. Since C++23, "deducing this" covers many CRTP use cases with less ceremony.
