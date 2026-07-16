## challenge: count live instances per type with CRTP
tags: templates, crtp
track: core
difficulty: medium

Write a CRTP base `Counted<Derived>` that tracks how many instances of each derived class are currently alive. Requirements: a static `alive()` returning the current count; the default constructor and the copy constructor both increment; the destructor decrements. The crucial property: `struct Widget : Counted<Widget>` and `struct Gadget : Counted<Gadget>` must each get an independent counter.

hint: The base is a class template over the derived type: `template <class Derived> class Counted`. Every instantiation is a distinct class — which is exactly what makes the counters independent.
hint: A static data member of a class template is per-instantiation, so `Counted<Widget>::count_` and `Counted<Gadget>::count_` are different variables. `inline static int count_ = 0;` avoids an out-of-line definition.
hint: Count every way an object can be born or die: default ctor `++`, copy ctor `++` (a copy is a new instance — forgetting this is the classic bug), dtor `--`. Copy *assignment* changes nothing: no object is created or destroyed.

```cpp
// starter
// Counted<Derived>: static alive() -> current live instance count.
// Default ctor and copy ctor increment; dtor decrements.
// Widget and Gadget below must each have their own counter.
// TODO: write the CRTP base class template.
```

```cpp
template <class Derived>
class Counted {
public:
    static int alive() { return count_; }
protected:
    Counted() { ++count_; }
    Counted(const Counted&) { ++count_; }
    ~Counted() { --count_; }
private:
    inline static int count_ = 0;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
struct Widget : Counted<Widget> {};
struct Gadget : Counted<Gadget> {};
int main() {
    assert(Widget::alive() == 0 && Gadget::alive() == 0);
    Widget a, b;
    assert(Widget::alive() == 2);
    assert(Gadget::alive() == 0);          // independent counter per derived type
    {
        Gadget g;
        Widget c(a);                       // copies count too
        assert(Widget::alive() == 3);
        assert(Gadget::alive() == 1);
    }
    assert(Widget::alive() == 2);          // scope exit ran the destructors
    assert(Gadget::alive() == 0);
    std::puts("PASS");
}
```

**Editorial:** The counter works because a class template's static members are *per instantiation*: `Counted<Widget>` and `Counted<Gadget>` are two unrelated classes, each with its own `count_` — a plain non-template base would lump every derived type into one number. That is the CRTP dividend: the derived type parameterizes the base, so type-keyed state falls out for free, with no map lookups and no RTTI. The details are where the marks are earned. The copy constructor must increment — if you omit it, the compiler-generated copy constructor is used, the count is not bumped, and the destructor of the copy later drives the counter negative. Copy assignment is correctly left alone since assignment neither creates nor destroys an instance. The constructors and destructor are `protected`, declaring that `Counted` is a mixin, not a standalone object — and the destructor is deliberately non-virtual, which is safe here precisely because nobody can `delete` through a `Counted<T>*`. Interestingly, this base never even calls `static_cast<Derived*>(this)`: sometimes CRTP's value is purely in the per-type instantiation.
