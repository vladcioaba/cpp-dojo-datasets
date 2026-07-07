## fact: std::move doesn't move anything
tags: move, core

`std::move(x)` is just a cast to rvalue reference — `static_cast<T&&>(x)`. It moves nothing; it only marks `x` as "safe to steal from". The actual move happens in whatever move constructor or move assignment receives it. If nothing receives it, nothing happens.

Corollary: `std::move` on a `const` object silently copies — the move constructor can't bind to `const T&&`... but the copy constructor can.
