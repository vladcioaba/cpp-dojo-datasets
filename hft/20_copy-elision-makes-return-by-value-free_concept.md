## fact: Copy elision makes return-by-value free
tags: rvo, copy-elision, cpp17
track: hft

Returning a local or temporary by value doesn't copy or move when elision applies — the object is built directly in the caller's storage. Since **C++17, copy elision is guaranteed** for returning a prvalue (`return T{...};`), even if the copy/move constructor is deleted. **NRVO** (returning a named local, `return x;`) is still only *permitted*, not mandated — but every serious compiler does it.

So `T make() { return T{...}; }` is genuinely zero-cost; you need no out-parameter and no `std::move` on the return. In fact `return std::move(local);` **pessimizes** — it forces a move and disables NRVO because it's no longer the plain named-return form. Return the name directly.
