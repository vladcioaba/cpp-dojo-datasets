## quiz: What is the cost model of C++ exceptions on the hot path?
tags: exceptions, hot-path, performance
track: hft

- [ ] `try`/`catch` adds overhead to every call even when nothing throws
- [x] With the table-based ("zero-cost") model the non-throwing path is essentially free, but an actual `throw` is very slow and non-deterministic (unwinding, RTTI) — so avoid throwing on the hot path
- [ ] Exceptions are faster than error codes on the happy path
- [ ] `-fno-exceptions` makes throwing faster

> Modern implementations use side tables to drive unwinding, so entering a `try` block costs nothing at runtime — the happy path is as fast as no exceptions at all. The price is paid only when you actually `throw`: the runtime walks unwind tables, runs destructors, and matches handlers, which is slow and has high variance — poison for tail latency. `-fno-exceptions` removes exception support entirely (shrinks the binary, forbids throwing code); it does not speed up throws.
