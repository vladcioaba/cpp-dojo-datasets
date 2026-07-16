## challenge: fix: the parent-child pair that never dies
tags: smart-pointers, raii
track: core
difficulty: easy

This code review found a leak: a parent node and its child are both managed by `shared_ptr`, but once the caller drops the family, *neither destructor ever runs* and memory climbs forever. Keep the back-pointer functional — `parentOf` must still find a child's parent — but make the pair actually die. The harness counts live nodes.

hint: Draw the ownership arrows. parent owns child (shared_ptr), child owns parent (shared_ptr) — each keeps the other's strong count at 1 even after every outside handle is gone.
hint: Owning edges must not form a loop. The parent -> child edge is real ownership; the child -> parent edge is only navigation. One of them should stop owning.
hint: Make `parent` a `std::weak_ptr<Node>` — assignment from a shared_ptr works unchanged — and have `parentOf` return `n.parent.lock()`.

```cpp
// starter
struct Node {
    inline static int alive = 0;
    std::shared_ptr<Node> child;
    std::shared_ptr<Node> parent;   // owning back-edge — completes a cycle
    Node() { ++alive; }
    ~Node() { --alive; }
};

// Builds: parent <-> child, returns the parent.
std::shared_ptr<Node> makeFamily() {
    auto parent = std::make_shared<Node>();
    auto child = std::make_shared<Node>();
    parent->child = child;
    child->parent = parent;
    return parent;
}

// Navigation helper: a node's parent, or nullptr at the root.
std::shared_ptr<Node> parentOf(const Node& n) {
    return n.parent;
}
```

```cpp
struct Node {
    inline static int alive = 0;
    std::shared_ptr<Node> child;    // ownership points DOWN the tree
    std::weak_ptr<Node> parent;     // navigation points up — non-owning
    Node() { ++alive; }
    ~Node() { --alive; }
};

// Builds: parent <-> child, returns the parent.
std::shared_ptr<Node> makeFamily() {
    auto parent = std::make_shared<Node>();
    auto child = std::make_shared<Node>();
    parent->child = child;
    child->parent = parent;         // shared -> weak assignment, unchanged
    return parent;
}

// Navigation helper: a node's parent, or nullptr at the root.
std::shared_ptr<Node> parentOf(const Node& n) {
    return n.parent.lock();
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    {
        auto family = makeFamily();
        assert(Node::alive == 2);

        // The back-pointer must still navigate correctly.
        assert(family->child != nullptr);
        assert(parentOf(*family->child).get() == family.get());
        assert(parentOf(*family) == nullptr);   // root has no parent
    }
    // Once the last outside handle is gone, BOTH nodes must be destroyed.
    // A shared_ptr cycle keeps alive == 2 here forever.
    assert(Node::alive == 0);
    std::puts("PASS");
}
```

**Editorial:** Reference counting has exactly one blind spot, and this is it. When `family` goes out of scope the parent's strong count drops to 1 — the child's `parent` member still owns it — and the child's count is 1 via the parent's `child` member. Two objects, each waiting for the other's destructor to release it: nothing dangles, nothing crashes, the pair is simply unreachable and immortal. `Node::alive` stays 2, which is why leak checks in tests are worth their weight. The fix is a design rule, not a trick: decide which edges *own* and make every other edge an observer. In a tree, ownership flows downward — parents own children — so the upward pointer becomes `weak_ptr<Node>`. Construction code doesn't change (assigning a `shared_ptr` into a `weak_ptr` is implicit), and navigation becomes `parent.lock()`, which returns an owning handle while you use it or `nullptr` if the parent died first — the root case falls out for free from a default-constructed `weak_ptr`. The same shape recurs everywhere `shared_ptr` is used bidirectionally: observers pointing back at subjects, children at parents, cache entries at caches. If two `shared_ptr`s can reach each other, one of them is a bug.
