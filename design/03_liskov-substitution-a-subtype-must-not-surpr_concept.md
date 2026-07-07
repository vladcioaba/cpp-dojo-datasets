## fact: Liskov Substitution — a subtype must not surprise the caller
tags: solid, lsp, inheritance
track: design

The **Liskov Substitution Principle**: code written against a base type must keep working when handed any derived type. A subtype may not strengthen preconditions or weaken postconditions the caller relies on. The canonical violation is `Square : Rectangle` — a caller that sets width and height independently (a reasonable `Rectangle` contract) gets a broken `Square`.

The fix is *not* a cleverer override — it is recognizing they are not substitutable. A square **is-not-a** mutable rectangle. Model them separately, or relate them only through a read-only interface where the offending mutators don't exist.

```cpp
struct Rectangle {
    virtual void set_width(int w)  { w_ = w; }
    virtual void set_height(int h) { h_ = h; }
    int area() const { return w_ * h_; }
protected: int w_{}, h_{};
};
struct Square : Rectangle {                    // LSP violation
    void set_width(int w)  override { w_ = h_ = w; }   // silently mutates height too
    void set_height(int h) override { w_ = h_ = h; }   // breaks assert(area == w*h)
};
// Fix: no inheritance link. Separate value types, or a shared `const` Shape view.
```
