## fact: volatile is for MMIO, not threads
tags: volatile, atomics, concurrency, trap
track: hft

A classic interview trap. `volatile` tells the compiler "this memory can change outside the program, don't optimize the access away" — it stops the compiler eliding/reordering *volatile* accesses. It does **not** provide atomicity, does **not** establish inter-thread ordering, and emits **no CPU fences**. Using `volatile` for thread communication is a data race (UB) and broken on any weakly-ordered CPU.

`volatile`'s real jobs are **memory-mapped I/O** (hardware registers), signal handlers (`volatile sig_atomic_t`), and `setjmp` locals. For thread-to-thread communication use `std::atomic`, which gives atomicity *and* the memory-order guarantees `volatile` lacks. Slogan: "`volatile` for hardware, `atomic` for threads."
