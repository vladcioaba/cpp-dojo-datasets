## quiz: Spinlock vs `std::mutex` for a very short critical section
tags: concurrency, spinlock, mutex, latency
track: hft

- [ ] Always use `std::mutex`; spinlocks are deprecated
- [x] For an extremely short critical section a spinlock can win by staying in userspace (no syscall/context switch), but it burns a core and degrades badly under contention or oversubscription
- [ ] A spinlock is always faster than a mutex
- [ ] `std::mutex` never makes a system call

> An uncontended `std::mutex` lock is typically a fast userspace atomic, but on contention it futex-sleeps — a syscall plus a context switch, which is terrible for tail latency. A spinlock keeps the waiter in userspace, which is a clear win when the holder releases within nanoseconds and the waiter has a dedicated core (add `_mm_pause()` in the loop to be hyperthread-friendly). But if the holder is descheduled — common under oversubscription — the spinner wastes an entire time slice, so a spinlock is not unconditionally faster.
