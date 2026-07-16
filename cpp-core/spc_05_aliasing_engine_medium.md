## challenge: hand out a member with the aliasing constructor
tags: smart-pointers, raii
track: core
difficulty: medium

A `Car` owns its `Engine` by value — the engine is a member, not a separate allocation. Implement `engineOf`, which returns a `std::shared_ptr<Engine>` pointing at `car->engine` that *shares ownership of the whole Car*: as long as anyone holds the engine pointer, the car must stay alive, and the car is destroyed only when the last engine handle drops. The harness tracks the Car's lifetime to verify exactly that.

hint: `std::shared_ptr<Engine>(&car->engine)` compiles and is a disaster: a brand-new control block now thinks it owns a raw `Engine*` into the middle of a Car, and will `delete` it. You need to share the CAR's control block.
hint: This is the aliasing constructor: `std::shared_ptr<M>(owner, ptr)` — first argument says whose lifetime to share, second says where to point. The two are allowed to differ; that's its entire purpose.
hint: `return std::shared_ptr<Engine>(car, &car->engine);` — use_count of the result counts owners of the Car's control block.

```cpp
// starter
struct Engine {
    int horsepower = 90;
};

struct Car {
    inline static int alive = 0;
    Engine engine;                  // owned by value — not its own allocation
    Car() { ++alive; }
    ~Car() { --alive; }
};

// TODO: return a shared_ptr that POINTS at car->engine but shares ownership
// of the whole Car, keeping it alive as long as the engine is referenced.
std::shared_ptr<Engine> engineOf(const std::shared_ptr<Car>& car) {
    return {};
}
```

```cpp
struct Engine {
    int horsepower = 90;
};

struct Car {
    inline static int alive = 0;
    Engine engine;                  // owned by value — not its own allocation
    Car() { ++alive; }
    ~Car() { --alive; }
};

std::shared_ptr<Engine> engineOf(const std::shared_ptr<Car>& car) {
    // Aliasing constructor: share ownership of *car, point at car->engine.
    return std::shared_ptr<Engine>(car, &car->engine);
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::shared_ptr<Engine> e;
    {
        auto car = std::make_shared<Car>();
        assert(Car::alive == 1);

        e = engineOf(car);
        assert(e != nullptr);
        assert(e.get() == &car->engine);    // points INTO the car
        assert(e->horsepower == 90);
        assert(e.use_count() == 2);         // e and car share ONE control block
        assert(car.use_count() == 2);
    }                                       // last Car-typed handle gone...
    assert(Car::alive == 1);                // ...but the engine handle keeps
    assert(e->horsepower == 90);            //    the whole Car alive
    e.reset();                              // last owner drops
    assert(Car::alive == 0);                // now the Car is destroyed
    std::puts("PASS");
}
```

**Editorial:** A `shared_ptr` carries two pointers: the one you see (`get()`) and the control block that decides lifetime. Normally they refer to the same object, but the aliasing constructor `shared_ptr<M>(owner, ptr)` deliberately splits them: the result co-owns whatever `owner` owns, while pointing wherever `ptr` says. That is precisely the shape of "hand out a piece of a shared aggregate": the engine is a by-value member with no allocation of its own, so its lifetime *is* the car's, and the only correct way to share it is to share the car. The harness pins the semantics down: after the local `car` handle dies, `Car::alive` is still 1 — the engine handle alone holds the car — and only `e.reset()` finally destroys it. Also note `e.use_count() == 2`: use_count reports owners of the shared control block, regardless of what the pointer aims at. The naive `std::shared_ptr<Engine>(&car->engine)` creates a *second* control block owning a pointer into the middle of another object; when it drains, it calls `delete` on memory that was never allocated alone — heap corruption. Aliasing shows up all over real code: exposing a field of a shared config, a node of a shared document tree, or a subobject of a pooled resource.
