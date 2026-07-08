## challenge: Ticket lock
tags: lock-free, ticket-lock, fairness
track: hft
difficulty: medium

A **fair** spinlock that hands out service in FIFO order — no thread starves, unlike the test-and-set spinlock. Keep two counters: `next_` (the next ticket to dispense) and `serving_` (the ticket currently allowed in). `lock()` atomically takes a ticket with `next_.fetch_add(1)` then spins until `serving_ == my_ticket`. `unlock()` bumps `serving_` so the next ticket-holder proceeds. Add a `try_lock()` that succeeds only when the lock is completely free (`serving_ == next_`) and no one is queued. Single-threaded correctness only; the harness drives the ticket/serving state machine.

Constraints: `lock()` must grab its ticket with one atomic RMW. `try_lock()` must not advance `next_` unless it actually acquires.

Example: from a fresh lock, `try_lock()` grabs it and `next_` becomes 1 while `serving_` stays 0; a second `try_lock()` (held) fails; after `unlock()`, `serving_` is 1 and a `lock()` taking ticket 1 proceeds immediately.

hint: The ticket you hold is the value `fetch_add` returns — a unique, monotonically increasing integer per caller.
hint: The lock is free exactly when `serving_ == next_` (nobody dispensed and unserved). That is the only state where `try_lock()` may claim it.
hint: `unlock()` publishes the critical section with a `release` store/RMW on `serving_`; waiters `acquire`-load `serving_` so they see that work.

```cpp
// starter
#include <atomic>
struct TicketLock {
    std::atomic<unsigned> next_{0};      // next ticket to hand out
    std::atomic<unsigned> serving_{0};   // ticket being served now
    void lock();
    bool try_lock();
    void unlock();
};
```

```cpp
void lock() {
    unsigned my = next_.fetch_add(1, std::memory_order_relaxed);
    while (serving_.load(std::memory_order_acquire) != my) { /* spin */ }
}
bool try_lock() {
    unsigned s = serving_.load(std::memory_order_relaxed);
    unsigned n = next_.load(std::memory_order_relaxed);
    if (s != n) return false;            // held or someone queued
    return next_.compare_exchange_strong(n, n + 1,
               std::memory_order_acquire, std::memory_order_relaxed);
}
void unlock() {
    serving_.fetch_add(1, std::memory_order_release);
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
struct TicketLock {
    std::atomic<unsigned> next_{0};
    std::atomic<unsigned> serving_{0};
    //__USER__
};
int main() {
    TicketLock t;
    if (!t.try_lock()) { std::puts("try_lock on free must succeed"); return 1; }
    if (t.next_.load() != 1 || t.serving_.load() != 0) { std::puts("counters after grab"); return 1; }
    if (t.try_lock()) { std::puts("try_lock on held must fail"); return 1; }
    t.unlock();
    if (t.serving_.load() != 1) { std::puts("serving after unlock"); return 1; }
    t.lock();
    if (t.next_.load() != 2) { std::puts("next after lock"); return 1; }
    if (t.try_lock()) { std::puts("try_lock on held 2"); return 1; }
    t.unlock();
    for (int i = 0; i < 1000; ++i) {
        t.lock();
        if (t.try_lock()) { std::puts("stress held"); return 1; }
        t.unlock();
    }
    if (t.next_.load() != t.serving_.load()) { std::puts("must be balanced"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The ticket lock trades the spinlock's unfairness for FIFO service by turning acquisition into "take a number, wait for it to be called." `next_.fetch_add(1)` is a single atomic RMW that hands every caller a distinct, strictly increasing ticket, so ordering is decided at the moment you enter the queue — impossible with a bare test-and-set where the winner is whoever the cache-coherence protocol happens to favor. A caller spins reading `serving_` (a cheap load, not a CAS, so under contention only one line is written per handoff), and `unlock()` increments `serving_`, releasing exactly the holder of the next ticket. Minimal-correct ordering: `next_.fetch_add` can be `relaxed` (it only needs uniqueness, not publication), the spin load on `serving_` is `acquire`, and `unlock`'s increment is `release` — that release/acquire pair transfers the critical section from holder to successor. `try_lock` refuses unless `serving_ == next_`, the only genuinely-free state, and uses a CAS so it can't steal a ticket someone else is about to claim.
