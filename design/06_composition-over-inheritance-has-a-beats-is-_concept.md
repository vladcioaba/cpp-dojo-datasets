## fact: Composition over inheritance — has-a beats is-a
tags: composition, inheritance, oop
track: design

Inheritance for code reuse creates a rigid, fan-out taxonomy: the moment behaviors combine, you get a combinatorial explosion (`FlyingBird`, `SwimmingBird`, `FlyingSwimmingBird`…) and a fixed hierarchy chosen at compile time. **Composition** assembles behavior from independent, swappable parts held as members — configurable at runtime, and free of the fragile-base-class problem.

Rule of thumb: use inheritance only for genuine *is-a substitutability* (LSP holds); use composition for *has-a* and for reuse.

```cpp
// Inheritance explodes as capabilities multiply. Compose them instead:
struct FlyBehavior  { std::function<void()> fly; };
struct SwimBehavior { std::function<void()> swim; };

class Duck {
    FlyBehavior  wings_;   // has-a
    SwimBehavior feet_;    // has-a
public:
    Duck(FlyBehavior f, SwimBehavior s) : wings_(std::move(f)), feet_(std::move(s)) {}
    void migrate() { wings_.fly(); feet_.swim(); }   // behavior composed, swappable at runtime
};
```
