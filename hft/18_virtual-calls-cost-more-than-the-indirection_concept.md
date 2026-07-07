## fact: Virtual calls cost more than the indirection
tags: virtual, vtable, inlining
track: hft

A virtual call adds a load (fetch the vtable pointer, then the function pointer) and an indirect branch the CPU may mispredict. But the bigger cost is what it **prevents**: the compiler usually can't see the target, so it **can't inline**, blocking constant propagation, vectorization, and cross-call optimization.

Hot-path alternatives: **CRTP** (static polymorphism), templates, `std::variant` + `std::visit`, or a plain `switch` on a type tag. If you must dispatch dynamically, keeping the same type in a tight loop lets the indirect-branch predictor learn it, and `final` can let the compiler **devirtualize** when the dynamic type is provable.
