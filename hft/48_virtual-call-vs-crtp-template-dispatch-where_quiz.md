## quiz: Virtual call vs CRTP/template dispatch — where does the cost come from?
tags: virtual, crtp, inlining
track: hft

- [ ] A virtual call is exactly as fast as a direct call once the vtable is in cache
- [x] A virtual call is an indirect call through the vtable that usually cannot be inlined, which blocks downstream optimizations; CRTP/templates bind the call at compile time so it inlines
- [ ] CRTP is slower because template instantiation always bloats the i-cache and loses
- [ ] Virtual calls are cheap; the real cost is the virtual destructor

> The vtable load and indirect branch are cheap in isolation; the real cost is that the compiler cannot see the callee through an indirect call, so it cannot inline it or propagate constants across the boundary — and it may eat a branch mispredict or i-cache miss on the target. CRTP (or plain templates) resolves the concrete type at compile time, so the call is direct and inlinable. Code bloat from many instantiations is a genuine trade-off, but on the hot path the inlining win usually dominates.
