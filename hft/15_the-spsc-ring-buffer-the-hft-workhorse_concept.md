## fact: The SPSC ring buffer, the HFT workhorse
tags: spsc, ring-buffer, lock-free
track: hft

A single-producer/single-consumer ring buffer needs **no locks and no CAS**: only one thread writes `head`, only one writes `tail`. Correctness comes purely from **release/acquire pairing** — the producer fills the slot, then release-stores the new head; the consumer acquire-loads head and is guaranteed to see the slot's data.

Keep `head` and `tail` on **separate cache lines** or the producer and consumer false-share the control indices. Size a power of two so wrap is a mask, not a modulo. Each side caches the other's index to avoid re-reading the contended atomic every iteration.

```cpp
alignas(64) std::atomic<size_t> head{0}; // written by producer only
alignas(64) std::atomic<size_t> tail{0}; // written by consumer only
// producer: buf[h & mask] = x; head.store(h + 1, std::memory_order_release);
// consumer: if (t != head.load(std::memory_order_acquire)) {
//               x = buf[t & mask]; tail.store(t + 1, std::memory_order_release); }
```
