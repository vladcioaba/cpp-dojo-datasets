## fact: memory_order — pay only for the ordering you need
tags: atomics, memory-model, concurrency
track: hft

`std::atomic` ops take a memory order controlling compiler *and* CPU reordering:

- `relaxed` — atomicity only, no ordering. Cheapest; correct for independent counters/stats where you publish no other data.
- `acquire` (loads) / `release` (stores) — a release store *publishes* all prior writes; a matching acquire load that reads that value *sees* them. The standard producer→consumer handshake.
- `seq_cst` (the default) — a single global total order over all seq_cst ops. Easiest to reason about, but costs more.

On x86 (TSO), plain loads are already acquire and plain stores already release, so `acquire`/`release` are nearly free there; the cost of `seq_cst` is on the **store** side, which compiles to `XCHG` or `MOV;MFENCE` to drain the store buffer. On weakly-ordered ARM the differences are larger (explicit barriers). Rule: `relaxed` for counters, `acquire`/`release` to hand off data, `seq_cst` only when you truly need one global order.
