## challenge: Seqlock reader/writer
tags: lock-free, seqlock, versioning
track: hft
difficulty: hard

A **seqlock** lets one writer publish a small struct (a price + size, a timestamp pair) to many readers with *no reader-side writes and no reader ever blocking the writer* — ideal for a market-data snapshot read far more often than updated. A single `std::atomic<unsigned> seq_` versions the payload. The writer bumps `seq_` to **odd** (signalling "update in progress"), stores the fields, then bumps it to **even** (signalling "stable"). A reader snapshots `seq_`, copies the fields, re-reads `seq_`, and retries if the version changed or was odd — guaranteeing it never returns a torn (half-updated) pair. Implement `void write(long a, long b)` and `void read(long& a, long& b) const`. Single-threaded correctness; the harness drives the version protocol.

Constraints: readers must not write shared state. The version must go odd during a write and even when stable; a read must retry on an odd or changed version.

Example: after `write(i, 2*i)` the version is even and equals `2*i`; a `read` returns exactly the pair `(i, 2*i)` — never a mix of two different writes.

hint: The writer does two `seq_` stores around the data writes: `seq_+1` (odd) before, `seq_+2` (even) after. A stable version is always even.
hint: The reader loops: read `s0`, copy the fields, read `s1`; accept only when `s0 == s1` and `s0` is even, otherwise a write overlapped — retry.
hint: The writer's second store is `release` and the reader's re-read is `acquire` (with an `acquire` fence after copying), so the field reads can't be reordered past the version check.

```cpp
// starter
#include <atomic>
struct SeqLock {
    std::atomic<unsigned> seq_{0};   // even = stable, odd = write in progress
    long a_ = 0;
    long b_ = 0;
    void write(long a, long b);
    void read(long& a, long& b) const;
};
```

```cpp
void write(long a, long b) {
    unsigned s = seq_.load(std::memory_order_relaxed);
    seq_.store(s + 1, std::memory_order_relaxed);        // enter: odd
    std::atomic_thread_fence(std::memory_order_release); // fields land after the odd marker
    a_ = a;
    b_ = b;
    seq_.store(s + 2, std::memory_order_release);        // leave: even, publish
}
void read(long& a, long& b) const {
    unsigned s0, s1;
    do {
        s0 = seq_.load(std::memory_order_acquire);
        a = a_;
        b = b_;
        std::atomic_thread_fence(std::memory_order_acquire);
        s1 = seq_.load(std::memory_order_relaxed);
    } while ((s0 & 1u) || s0 != s1);   // retry if writing, or a write overlapped
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
struct SeqLock {
    std::atomic<unsigned> seq_{0};
    long a_ = 0;
    long b_ = 0;
    //__USER__
};
int main() {
    SeqLock sl;
    long a, b;
    sl.read(a, b);
    if (a != 0 || b != 0) { std::puts("initial read"); return 1; }
    for (long i = 1; i <= 1000; ++i) {
        sl.write(i, i * 2);
        if (sl.seq_.load() & 1u) { std::puts("seq must be even after write"); return 1; }
        if (sl.seq_.load() != (unsigned)(2 * i)) { std::puts("seq must += 2 per write"); return 1; }
        sl.read(a, b);
        if (a != i || b != 2 * i) { std::puts("incoherent pair"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** A seqlock inverts the usual read/write cost model: writers are cheap and never wait for readers, readers pay nothing but a version check and *may* retry, and — crucially — readers issue no stores, so the payload cache line stays in shared state and isn't ping-ponged by reader-side lock acquisition. That is exactly what you want for a hot, read-mostly snapshot. The protocol is the odd/even sequence counter: an odd `seq_` means "in flight," so a reader that samples an odd value, or sees `seq_` change between its two reads, knows a writer overlapped its copy and discards the possibly-torn result. Because the reader reads the plain `a_`/`b_` fields *between* two atomic reads of `seq_`, ordering is essential: the writer's final store is `release` and the reader's version reads are `acquire`, and the `acquire` fence after copying the fields prevents the compiler or CPU from hoisting those field reads below the second `seq_` load — without it the check would be meaningless. The odd-marking store can be `relaxed` paired with a `release` fence before the data. Two caveats the single-threaded harness can't exercise: the payload is formally a data race in a real multi-threaded run (readers may observe a half-written value, which the retry then throws away — well-defined only under the seqlock idiom, not the abstract C++ memory model), and a reader can livelock under a relentlessly-writing writer, so seqlocks suit *infrequent* updates.
