## fact: Do it at compile time
tags: constexpr, templates, compile-time
track: hft

Work done at compile time costs zero at run time. `constexpr` (and C++20 `consteval`/`constinit`) evaluates during compilation — precompute lookup tables, parse config, validate constants — so the hot path just reads a baked-in result with no static-init overhead on the fast path.

Templates specialize code per type, so there's **no runtime dispatch**: the compiler emits and inlines a dedicated version, enabling constant folding and vectorization a virtual call would block. Interviewers like to see branching moved to compile time — e.g. templating a strategy on order-type flags so the hot loop has no `if`. The cost is compile time and code bloat (I-cache pressure), so specialize the hot path, not everything.
