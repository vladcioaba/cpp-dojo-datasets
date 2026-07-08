## challenge: Treiber lock-free stack
tags: lock-free, stack, cas
track: hft
difficulty: medium

The canonical lock-free container: a **Treiber stack**, a singly-linked LIFO whose only shared state is an `std::atomic<Node*> head_`. `push(v)` allocates a node, points its `next` at the current head, and CAS-swaps the head to the new node. `pop(out)` reads the head; if null the stack is empty, otherwise CAS the head to `head->next` and hand back the popped value. Both use a CAS retry loop so a concurrent modification just re-reads and tries again. Single-threaded correctness only; the harness checks LIFO ordering and empty handling.

Constraints: the only synchronization is CAS on `head_`. `pop` on an empty stack returns `false`.

Example: push 0,1,2,3,4 then pop five times yields 4,3,2,1,0 (LIFO); popping the now-empty stack returns `false`.

hint: `push` sets `n->next = head` then loops `compare_exchange_weak(n->next, n)` — on failure the CAS refreshes `n->next` to the new head for free, so you just retry.
hint: `pop` must load `old = head`, check for null, then CAS `head` from `old` to `old->next`; loop while the CAS fails.
hint: Publishing the node needs `release` on the successful `push` CAS; a popper `acquire`-loads the head so it sees the fully-written node.

```cpp
// starter
#include <atomic>
template <class T>
struct TreiberStack {
    struct Node { T val; Node* next; };
    std::atomic<Node*> head_{nullptr};
    void push(const T& v);
    bool pop(T& out);
};
```

```cpp
void push(const T& v) {
    Node* n = new Node{v, head_.load(std::memory_order_relaxed)};
    while (!head_.compare_exchange_weak(n->next, n,
               std::memory_order_release, std::memory_order_relaxed)) {}
    // on failure, n->next was refreshed to the current head; retry
}
bool pop(T& out) {
    Node* old = head_.load(std::memory_order_acquire);
    while (old && !head_.compare_exchange_weak(old, old->next,
               std::memory_order_acquire, std::memory_order_relaxed)) {}
    if (!old) return false;
    out = old->val;
    delete old;
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
template <class T>
struct TreiberStack {
    struct Node { T val; Node* next; };
    std::atomic<Node*> head_{nullptr};
    //__USER__
};
int main() {
    TreiberStack<int> s;
    int x = 0;
    if (s.pop(x)) { std::puts("pop on empty must fail"); return 1; }
    for (int i = 0; i < 5; ++i) s.push(i);
    for (int i = 4; i >= 0; --i) {
        if (!s.pop(x) || x != i) { std::puts("LIFO order broken"); return 1; }
    }
    if (s.pop(x)) { std::puts("empty again must fail"); return 1; }
    s.push(10); s.push(20);
    if (!s.pop(x) || x != 20) { std::puts("i1"); return 1; }
    s.push(30);
    if (!s.pop(x) || x != 30) { std::puts("i2"); return 1; }
    if (!s.pop(x) || x != 10) { std::puts("i3"); return 1; }
    if (s.pop(x)) { std::puts("i4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The Treiber stack shows the CAS-retry pattern in its purest form: read the head, build the intended new head, and atomically swap only if the head hasn't moved — otherwise loop. The elegance is that `compare_exchange_weak(expected, desired)` writes the *observed* value back into `expected` on failure, so `push`'s retry automatically re-links the new node onto whatever the head became. Minimal-correct ordering: the `push` success is `release` so the node's fields are visible before its pointer is published; `pop` `acquire`-loads (and acquires on its CAS) so a popper that observes a node also observes that node's contents — a classic release/acquire pair on `head_`. The failure legs only need `relaxed`. Two real-world caveats the single-threaded harness can't show: the **ABA problem** (head goes A→B→A between your load and CAS, fooling the compare) which production code fixes with tagged pointers or hazard pointers, and **safe reclamation** — you cannot simply `delete` a popped node while other threads may still hold it, so real implementations defer freeing. Here, single-threaded, `delete` is safe and LIFO ordering is exact.
