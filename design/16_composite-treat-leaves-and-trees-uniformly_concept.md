## fact: Composite — treat leaves and trees uniformly
tags: patterns, structural, composite, recursion
track: design

The **Composite** pattern lets a client treat an individual object and a group of objects through one interface. Both a `File` and a `Directory` are `Node`s; asking a directory for its `size()` recurses into children, but the caller writes the same call for either. It's the shape of every scene graph, GUI widget tree, and filesystem.

The recursion lives entirely inside the branch type — leaves stay trivial, and the client code has no `if (is_leaf)` anywhere.

```cpp
struct Node { virtual int size() const = 0; virtual ~Node() = default; };

struct File : Node { int bytes; int size() const override { return bytes; } };

struct Directory : Node {
    std::vector<std::unique_ptr<Node>> children;
    int size() const override {
        int total = 0;
        for (const auto& c : children) total += c->size();  // leaf and branch: identical call
        return total;
    }
};
```
