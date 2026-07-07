## fact: malloc is a latency landmine
tags: allocation, hot-path, memory
track: hft

`new`/`malloc` can take a lock (glibc arenas), walk a free-list, fault in a fresh page, or drop into the kernel (`mmap`/`brk`) — latency from tens of nanoseconds to microseconds, with an unbounded tail. Unacceptable on the hot path.

Techniques: **preallocate** everything at startup; use **object pools**/free-lists, **arena/bump allocators**, and fixed-capacity containers (`reserve()` up front, `std::array`, ring buffers). `std::pmr` (`monotonic_buffer_resource`) lets standard containers draw from a preallocated buffer. The goal is **zero allocation in steady state** — allocate before the open, never during.
