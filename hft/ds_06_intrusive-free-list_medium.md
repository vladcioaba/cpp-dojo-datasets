## challenge: Intrusive singly-linked free list
tags: allocation, free-list, intrusive, memory
track: hft
difficulty: medium

Node-based structures without a per-node allocation: pre-allocate `N` nodes in an array and thread a free list through a `next` index that each node already owns. The same `next` field links a node into the free list while it is idle and into a user list while it is in use — that is what "intrusive" means. Implement `size_t acquire()` (unlink and return a free node index, or `NIL` when exhausted) and `void release(size_t)` (push a node back onto the free list). Both are O(1) and never touch the allocator.

Constraints: `N` is a compile-time constant, `1 <= N`. `NIL` is a sentinel index. No heap allocation. `acquire`/`release` form a stack (a released node is the next one handed out — LIFO).

Example: on `FreeList<3>`, three `acquire()` calls return three distinct valid indices, the 4th returns `NIL`; after `release(b)`, the next `acquire()` returns `b` again; nodes can be chained via their `next` field into a user list and traversed without any allocation.

hint: Initialize the free list once in the constructor by linking node `i` to node `i+1`, with the last node's `next` set to `NIL` and a `free_head_` pointing at node 0.
hint: `acquire` pops the head: remember `free_head_`, advance it to that node's `next`, return the old head.
hint: `release` pushes onto the head: set the node's `next` to the current `free_head_`, then point `free_head_` at that node.

```cpp
// starter
template <size_t N>
struct FreeList {
    struct Node { int value; size_t next; };
    static constexpr size_t NIL = ~size_t(0);
    Node nodes_[N];
    size_t free_head_;
    FreeList() {
        for (size_t i = 0; i < N; ++i) nodes_[i].next = i + 1;
        nodes_[N - 1].next = NIL;
        free_head_ = 0;
    }
    // implement acquire / release
};
```

```cpp
size_t acquire() {
    if (free_head_ == NIL) return NIL;
    size_t i = free_head_;
    free_head_ = nodes_[i].next;
    return i;
}
void release(size_t i) {
    nodes_[i].next = free_head_;
    free_head_ = i;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <size_t N>
struct FreeList {
    struct Node { int value; size_t next; };
    static constexpr size_t NIL = ~size_t(0);
    Node nodes_[N];
    size_t free_head_;
    FreeList() {
        for (size_t i = 0; i < N; ++i) nodes_[i].next = i + 1;
        nodes_[N - 1].next = NIL;
        free_head_ = 0;
    }
    //__USER__
};
int main() {
    const size_t NIL = FreeList<3>::NIL;
    FreeList<3> fl;
    size_t a = fl.acquire();
    size_t b = fl.acquire();
    size_t c = fl.acquire();
    if (a == NIL || b == NIL || c == NIL) { std::puts("first 3 acquires must succeed"); return 1; }
    if (a == b || b == c || a == c) { std::puts("acquired nodes must be distinct"); return 1; }
    if (fl.acquire() != NIL) { std::puts("exhausted list must return NIL"); return 1; }
    // intrusive use: chain a -> b -> c through the same next field, then traverse
    fl.nodes_[a].value = 10; fl.nodes_[a].next = b;
    fl.nodes_[b].value = 20; fl.nodes_[b].next = c;
    fl.nodes_[c].value = 30; fl.nodes_[c].next = NIL;
    int sum = 0;
    for (size_t i = a; i != NIL; i = fl.nodes_[i].next) sum += fl.nodes_[i].value;
    if (sum != 60) { std::puts("intrusive chain traversal"); return 1; }
    fl.release(b);                              // b goes back to the free list
    size_t d = fl.acquire();
    if (d != b) { std::puts("released node must be reused (LIFO)"); return 1; }
    if (fl.acquire() != NIL) { std::puts("full again must return NIL"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** An intrusive free list is how you get node-based data structures (linked lists, trees, hash chains) with zero per-node allocation: the nodes live in one contiguous array and the `next` link is a field they already carry, doing double duty as the free-list pointer when idle and the user-list pointer when live. `acquire`/`release` are a pointer-free stack over that array — pop the head to allocate, push to free — so both are branch-light O(1). Versus `std::list` or a `new`-per-node scheme, you save an allocator round-trip on every insert, keep the nodes cache-adjacent (indices, not scattered heap addresses), and can even relocate the whole pool by copying the array, because links are indices rather than absolute pointers.
