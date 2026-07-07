## fact: Encapsulation is about invariants, not just private fields
tags: encapsulation, invariants, oop
track: design

Making data `private` is the mechanism; the *point* is protecting an **invariant** — a property that is always true of a valid object. The constructor **establishes** the invariant; every mutating method **preserves** it. If no operation can leave the object in an invalid state, the rest of your program never has to check.

Here every path that changes the value funnels through `clamp`, so `0 <= value() <= 100` holds for the object's entire lifetime — by construction.

```cpp
class Percentage {
    double v_;                         // invariant: 0.0 <= v_ <= 100.0
    static double clamp(double x) { return std::min(100.0, std::max(0.0, x)); }
public:
    explicit Percentage(double v) : v_(clamp(v)) {}        // established at birth
    void adjust(double delta) { v_ = clamp(v_ + delta); }  // preserved on every mutation
    double value() const { return v_; }
};
```
