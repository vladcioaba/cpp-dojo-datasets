## fact: shared_ptr's refcount is atomic — and not free
tags: shared-ptr, atomics, memory
track: hft

Copying a `std::shared_ptr` does an **atomic** increment of the control block's refcount; destroying one does an atomic decrement (with acquire/release ordering so the last owner deletes safely). Those atomic read-modify-writes are far pricier than a pointer copy, and if multiple threads touch the same control block they **contend** on that cache line.

On the hot path: pass `const shared_ptr&` or a raw/`T*` observer instead of copying; prefer `unique_ptr` (no refcount) or value semantics; don't churn shared_ptrs in a loop. Also, object and control block are two allocations unless you use `make_shared`, adding a pointer-chase. `weak_ptr::lock()` is atomic too. `shared_ptr` is a fine ownership tool — just not something to copy on every tick.
