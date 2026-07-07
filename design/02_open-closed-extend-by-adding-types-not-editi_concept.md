## fact: Open/Closed — extend by adding types, not editing switches
tags: solid, ocp, polymorphism
track: design

**Open for extension, closed for modification**: you should be able to add behavior without editing code that already works and is already tested. The tell-tale violation is a `switch`/`if` over a type tag that must grow a new branch every time a case is added — every extension re-opens (and risks re-breaking) that function.

Invert it with polymorphism: each new variant is a *new class* implementing an interface. Existing code calls the interface and never changes.

```cpp
// Closed to extension: every new shape edits this function.
double area(const ShapeData& s) {
    switch (s.kind) { case Circle: /*...*/; case Square: /*...*/; }  // grows forever
}

// Open: a new shape is a new class; the loop over Shape& never changes.
struct Shape { virtual double area() const = 0; virtual ~Shape() = default; };
struct Triangle : Shape { double b, h; double area() const override { return 0.5 * b * h; } };
```
