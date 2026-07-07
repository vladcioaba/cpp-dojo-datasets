## quiz: What mechanism does this code use?
tags: patterns, crtp, templates

```cpp
template <class D>
struct Counter {
    static inline int alive = 0;
    Counter() { ++alive; }
    ~Counter() { --alive; }
};
struct Widget : Counter<Widget> {};
struct Gadget : Counter<Gadget> {};
```

- [ ] Type erasure
- [ ] Virtual inheritance
- [x] CRTP — each derived class gets its own base instantiation and its own counter
- [ ] Dependency injection

> Curiously Recurring Template Pattern: `Counter<Widget>` and `Counter<Gadget>` are distinct types, so each derived class gets an independent `alive` counter — per-type behavior with zero runtime overhead.
