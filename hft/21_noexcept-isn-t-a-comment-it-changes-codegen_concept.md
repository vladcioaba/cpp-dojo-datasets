## fact: noexcept isn't a comment — it changes codegen
tags: noexcept, move, performance
track: hft

`noexcept` lets the optimizer skip unwinding paths around a call and is a **precondition for library fast paths**. The canonical case is `std::vector` growth: on reallocation it **moves** elements only if the move constructor is `noexcept`; otherwise it **copies**, to preserve the strong exception guarantee. A non-`noexcept` move ctor silently turns O(n) moves into O(n) copies on every regrowth.

`std::move_if_noexcept` and many `std::` operations branch on this. Mark move constructor, move assignment, and `swap` `noexcept` (destructors are `noexcept` by default). If a `noexcept` function does throw, `std::terminate` runs — so promise it only when true.
