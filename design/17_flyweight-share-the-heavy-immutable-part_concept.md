## fact: Flyweight — share the heavy immutable part
tags: patterns, structural, flyweight, memory
track: design

When you have millions of objects that differ only slightly, split their state: **intrinsic** state (shared, immutable — the species of a tree) versus **extrinsic** state (unique per instance — its x/y position). Store one shared copy of each distinct intrinsic value and point every object at it. A forest of a million trees needs only a handful of `TreeKind` objects.

An interning pool hands out shared, `const` handles, guaranteeing the heavy data exists once.

```cpp
struct TreeKind { std::string name; Texture bark; };   // intrinsic, shared, immutable

class TreeKindPool {
    std::unordered_map<std::string, std::shared_ptr<const TreeKind>> pool_;
public:
    std::shared_ptr<const TreeKind> get(const std::string& name);   // interned: one per distinct name
};

struct Tree { int x, y; std::shared_ptr<const TreeKind> kind; };    // extrinsic + shared handle
```
