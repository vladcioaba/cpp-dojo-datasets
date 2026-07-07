## quiz: What is wrong with this design?
tags: patterns, smart-pointers

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::shared_ptr<Node> prev;   // doubly linked
};
```

- [ ] shared_ptr is too slow for linked lists
- [x] next/prev cycles keep reference counts above zero — the list leaks
- [ ] Node needs a virtual destructor
- [ ] shared_ptr cannot point to an incomplete type

> Two nodes pointing at each other hold each other's count at ≥1 forever, so neither is ever destroyed. The fix is to break the cycle: make one direction non-owning, typically `std::weak_ptr<Node> prev;` (or raw pointer if lifetime is externally guaranteed).
