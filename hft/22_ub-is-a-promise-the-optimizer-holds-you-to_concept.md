## fact: UB is a promise the optimizer holds you to
tags: undefined-behavior, optimizer, aliasing
track: hft

The optimizer assumes UB **never happens** and transforms accordingly. Two big ones:

**Signed integer overflow is UB**, so the compiler assumes `x + 1 > x` always holds — it promotes loop counters, drops checks, and hoists code. This is usually a *win* (part of why `int` loop counters vectorize better than `unsigned`, which is defined to wrap). But relying on signed wraparound is a bug; use unsigned or `-fwrapv` for true modular arithmetic.

**Strict aliasing**: the compiler assumes pointers of *different* types don't alias (except `char*`), so it keeps values in registers across writes through an unrelated pointer. Reading a `float` through an `int*` therefore breaks at `-O2` — which is exactly why `reinterpret_cast` type-punning is UB.
