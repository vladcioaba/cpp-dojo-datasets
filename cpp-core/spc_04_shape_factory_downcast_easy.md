## challenge: factory returns unique_ptr, observers peek via get()
tags: smart-pointers, raii
track: core
difficulty: easy

Implement two functions around a small shape hierarchy: `makeShape` — a factory returning `std::unique_ptr<Shape>` owning the requested derived type — and `circleRadius`, which reports the radius *if* the pointee happens to be a `Circle`, without ever taking or transferring ownership. The harness checks the factory's products and that inspecting a shape leaves the caller's `unique_ptr` untouched.

hint: The factory: `std::make_unique<Circle>(dim)` converts implicitly to `std::unique_ptr<Shape>` — upcasting owning pointers is free. Unknown kinds return `nullptr` (a default-constructed unique_ptr).
hint: `circleRadius` only observes. Borrow the raw pointer with `s.get()` — do not `release()`, do not copy the unique_ptr (it won't compile, which is the type doing its job).
hint: Downcast the borrowed pointer with `dynamic_cast<const Circle*>(s.get())`; it yields `nullptr` for non-circles, which maps to the -1.0 return.

```cpp
// starter
struct Shape {
    virtual ~Shape() = default;
    virtual std::string name() const = 0;
};

struct Circle : Shape {
    double radius;
    explicit Circle(double r) : radius(r) {}
    std::string name() const override { return "circle"; }
};

struct Square : Shape {
    double side;
    explicit Square(double s) : side(s) {}
    std::string name() const override { return "square"; }
};

// TODO: "circle" -> owning pointer to Circle(dim); "square" -> Square(dim);
// anything else -> nullptr.
std::unique_ptr<Shape> makeShape(const std::string& kind, double dim) {
    return nullptr;
}

// TODO: if s currently points at a Circle, return its radius, else -1.0.
// Only LOOK at the object — ownership must stay with the caller.
double circleRadius(const std::unique_ptr<Shape>& s) {
    return -1.0;
}
```

```cpp
struct Shape {
    virtual ~Shape() = default;
    virtual std::string name() const = 0;
};

struct Circle : Shape {
    double radius;
    explicit Circle(double r) : radius(r) {}
    std::string name() const override { return "circle"; }
};

struct Square : Shape {
    double side;
    explicit Square(double s) : side(s) {}
    std::string name() const override { return "square"; }
};

std::unique_ptr<Shape> makeShape(const std::string& kind, double dim) {
    if (kind == "circle") return std::make_unique<Circle>(dim);
    if (kind == "square") return std::make_unique<Square>(dim);
    return nullptr;
}

double circleRadius(const std::unique_ptr<Shape>& s) {
    // Borrow, then downcast the borrowed pointer. Ownership never moves.
    if (const auto* c = dynamic_cast<const Circle*>(s.get())) {
        return c->radius;
    }
    return -1.0;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    auto c = makeShape("circle", 2.5);
    assert(c != nullptr);
    assert(c->name() == "circle");
    assert(circleRadius(c) == 2.5);
    assert(c != nullptr);               // inspection must not steal ownership
    assert(c->name() == "circle");      // object still alive and intact

    auto s = makeShape("square", 4.0);
    assert(s != nullptr);
    assert(s->name() == "square");
    assert(circleRadius(s) == -1.0);    // not a circle
    assert(s != nullptr);

    assert(makeShape("triangle", 1.0) == nullptr);

    // The factory's unique_ptr<Shape> still runs ~Circle: virtual dtor + RAII.
    std::puts("PASS");
}
```

**Editorial:** Two idioms meet here. The factory returns `std::unique_ptr<Shape>` by value: `make_unique<Circle>` produces a `unique_ptr<Circle>` that converts implicitly to `unique_ptr<Shape>` (owning pointers upcast as freely as raw ones), the return is a cheap pointer move, and the caller owns the result with zero ambiguity — this is the canonical modern factory signature, and it beats returning `shared_ptr` because a `unique_ptr` can always become shared later, never the reverse. Destruction stays correct because `Shape` has a virtual destructor; without it, deleting a `Circle` through `unique_ptr<Shape>` would be UB — smart pointers automate the *call* to delete, not its correctness. The second idiom is observation: `circleRadius` needs to look at the object, possibly as its dynamic type, but has no business owning it. `get()` is exactly this borrow — the raw pointer as a non-owning view — and `dynamic_cast` on that raw pointer gives the checked downcast, returning `nullptr` for non-circles. The tempting wrong moves both fail loudly: copying the `unique_ptr` doesn't compile, and `release()` would silently transfer ownership into a function that never deletes, leaking the shape. Raw pointers didn't die in modern C++; they just got demoted to observers.
