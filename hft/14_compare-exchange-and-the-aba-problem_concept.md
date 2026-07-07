## fact: compare_exchange and the ABA problem
tags: cas, lock-free, aba
track: hft

**CAS** underlies most lock-free code: `compare_exchange_strong(expected, desired)` atomically writes `desired` only if the value still equals `expected`, else reloads `expected`. Algorithms spin on this. Use `_weak` inside loops (may fail spuriously but cheaper on LL/SC machines); `_strong` when you don't already loop.

The **ABA problem**: a thread reads A, another changes it A→B→A, and the first thread's CAS succeeds because the value *looks* unchanged — even though the world moved (e.g. a freed-and-recycled node). Classic in lock-free stacks/queues with pointer reuse.

Fixes: a **tagged pointer / version counter** (pack a monotonic tag beside the pointer so "A with tag 1" ≠ "A with tag 2", often via double-width `cmpxchg16b`), **hazard pointers**, or **epoch-based reclamation** to stop memory being recycled underneath a reader.
