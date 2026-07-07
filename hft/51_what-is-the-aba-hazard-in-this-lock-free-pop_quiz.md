## quiz: What is the ABA hazard in this lock-free pop?
tags: concurrency, lock-free, cas, aba
track: hft

```cpp
Node* head = top.load();
while (!top.compare_exchange_weak(head, head->next)) {}
// ... return head;
```

- [ ] `compare_exchange_weak` can fail spuriously and corrupt the list
- [x] Between the load and the CAS, `head` can be popped and a *different* node reused at the same address; the pointer compares equal (A→B→A), so the CAS wrongly succeeds using a stale `head->next`
- [ ] CAS on a pointer is not atomic, so `head->next` can tear
- [ ] The loop can never terminate under contention

> The CAS only checks that the raw pointer bits still equal `head`. If another thread pops A, pops B, then pushes a freshly allocated node that the allocator happens to place at A's old address, the pointer looks unchanged even though `head->next` (read from a node that may already be freed/reused) is now stale — the CAS succeeds and corrupts the stack. Fixes: tagged/versioned pointers (a counter in spare bits), hazard pointers, or epoch/RCU reclamation. The spurious failure of the `weak` form is expected and simply retries — that is not the bug.
