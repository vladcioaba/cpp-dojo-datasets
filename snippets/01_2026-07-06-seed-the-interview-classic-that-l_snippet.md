## snippet: 2026-07-06 — seed — The interview classic that leaks
tags: smart-pointers, seed

```cpp
struct Cache {
    std::shared_ptr<Cache> parent;
    std::vector<std::shared_ptr<Cache>> children;
    void add(std::shared_ptr<Cache> c) {
        c->parent = shared_from_this();   // requires enable_shared_from_this
        children.push_back(std::move(c));
    }
};
```

**Analysis:** Parent owns children, children own parent — every parent/child pair forms a reference cycle, so the whole tree leaks when the last external `shared_ptr` drops. As written it's also UB: `shared_from_this()` requires inheriting `std::enable_shared_from_this<Cache>` *and* the object already being managed by a `shared_ptr`.

Fix: `std::weak_ptr<Cache> parent;` — children observe the parent, ownership flows one way (down). This exact shape shows up in GUI widget trees, scene graphs, and DOM-like structures.
