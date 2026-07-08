## challenge: Bounded MPSC queue (Vyukov cells)
tags: lock-free, mpsc, bounded-queue
track: hft
difficulty: hard

A bounded queue that many producers push to and (here) a consumer drains, using **per-cell sequence numbers** — Dmitry Vyukov's design, the workhorse behind many order-gateway queues. Each of `N` cells (N a power of two) carries an `std::atomic<size_t> seq`. Producers claim slots by CAS-advancing a shared `enq_` counter, but only when the target cell's `seq` equals the ticket — that gate is what makes it bounded and ABA-free. A cell is *ready to write* when `seq == pos`, *ready to read* when `seq == pos + 1`; after writing you set `seq = pos + 1`, after reading `seq = pos + N`. Initialize `buf_[i].seq = i`. Implement the constructor plus `bool push(const T&)` and `bool pop(T&)`. Single-threaded correctness; the harness checks FIFO, full/empty, and wraparound.

Constraints: `N` a power of two. `push` returns `false` when full, `pop` returns `false` when empty — detected via the signed `seq - pos` difference, never a separate size counter.

Example: with `N = 4`, four pushes fill it and the fifth fails; draining returns them in FIFO order; interleaving push/pop wraps around the ring indefinitely.

hint: `dif = (intptr_t)seq - (intptr_t)pos`. For `push`: `dif == 0` means the cell is yours to claim (CAS `enq_`), `dif < 0` means full, `dif > 0` means another producer moved on — reload `enq_` and retry.
hint: `pop` is symmetric against `pos + 1`: `dif == 0` claim it, `dif < 0` empty, `dif > 0` reload `deq_`.
hint: Read the cell's `seq` with `acquire` and publish the new `seq` with `release`; the `enq_`/`deq_` CAS itself can be `relaxed` because the per-cell `seq` carries the real handshake.

```cpp
// starter
#include <atomic>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <class T, size_t N>
struct MpscQueue {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    struct Cell { std::atomic<size_t> seq; T data; };
    Cell buf_[N];
    std::atomic<size_t> enq_{0};   // producer ticket
    std::atomic<size_t> deq_{0};   // consumer ticket
    MpscQueue();                   // seq[i] = i
    bool push(const T& v);
    bool pop(T& out);
};
```

```cpp
MpscQueue() {
    for (size_t i = 0; i < N; ++i)
        buf_[i].seq.store(i, std::memory_order_relaxed);
}
bool push(const T& v) {
    size_t pos = enq_.load(std::memory_order_relaxed);
    Cell* c;
    for (;;) {
        c = &buf_[pos & (N - 1)];
        size_t seq = c->seq.load(std::memory_order_acquire);
        std::intptr_t dif = (std::intptr_t)seq - (std::intptr_t)pos;
        if (dif == 0) {
            if (enq_.compare_exchange_weak(pos, pos + 1, std::memory_order_relaxed)) break;
        } else if (dif < 0) {
            return false;   // full
        } else {
            pos = enq_.load(std::memory_order_relaxed);
        }
    }
    c->data = v;
    c->seq.store(pos + 1, std::memory_order_release);   // mark readable
    return true;
}
bool pop(T& out) {
    size_t pos = deq_.load(std::memory_order_relaxed);
    Cell* c;
    for (;;) {
        c = &buf_[pos & (N - 1)];
        size_t seq = c->seq.load(std::memory_order_acquire);
        std::intptr_t dif = (std::intptr_t)seq - (std::intptr_t)(pos + 1);
        if (dif == 0) {
            if (deq_.compare_exchange_weak(pos, pos + 1, std::memory_order_relaxed)) break;
        } else if (dif < 0) {
            return false;   // empty
        } else {
            pos = deq_.load(std::memory_order_relaxed);
        }
    }
    out = c->data;
    c->seq.store(pos + N, std::memory_order_release);   // free for the next lap
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <cstddef>
#include <cstdint>
using std::size_t;
template <class T, size_t N>
struct MpscQueue {
    static_assert((N & (N - 1)) == 0, "N must be a power of two");
    struct Cell { std::atomic<size_t> seq; T data; };
    Cell buf_[N];
    std::atomic<size_t> enq_{0};
    std::atomic<size_t> deq_{0};
    //__USER__
};
int main() {
    MpscQueue<int, 4> q;
    int x = 0;
    if (q.pop(x)) { std::puts("pop on empty must fail"); return 1; }
    for (int i = 0; i < 4; ++i) if (!q.push(i)) { std::puts("must fit 4"); return 1; }
    if (q.push(99)) { std::puts("push on full must fail"); return 1; }
    for (int i = 0; i < 4; ++i) { if (!q.pop(x) || x != i) { std::puts("FIFO order broken"); return 1; } }
    if (q.pop(x)) { std::puts("empty again must fail"); return 1; }
    for (int r = 0; r < 100; ++r) { if (!q.push(r) || !q.pop(x) || x != r) { std::puts("wrap-around broken"); return 1; } }
    for (int i = 0; i < 3; ++i) q.push(i);
    if (!q.pop(x) || x != 0) { std::puts("p0"); return 1; }
    q.push(100);
    if (!q.pop(x) || x != 1) { std::puts("p1"); return 1; }
    if (!q.pop(x) || x != 2) { std::puts("p2"); return 1; }
    if (!q.pop(x) || x != 100) { std::puts("p3"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Vyukov's bounded queue is the design to reach for when you need multiple producers, a fixed footprint, and no dynamic allocation. Its trick is the per-cell sequence number, which encodes *whose turn it is* for each slot on each lap around the ring. The signed difference `seq - pos` is a three-way verdict: `0` means "this cell is exactly at the ticket you hold, claim it"; negative means "the cell is still owned by an earlier lap" — for a producer that's a full queue, for a consumer an empty one; positive means "another thread already advanced past here," so you reload the shared counter and retry. Producers race only on the `enq_` CAS; the loser simply re-reads and targets the next cell, so there is no lock and no unbounded retry storm. Crucially this sidesteps ABA without tagged pointers: a cell's `seq` only ever increases (by 1 when written, by `N` when consumed), so a stale observation can never masquerade as current. Minimal-correct ordering: the cell `seq` load is `acquire` and its store `release`, forming the real producer→consumer handshake that publishes `data`; the `enq_`/`deq_` counter CAS carries no payload and can stay `relaxed`. Single-threaded, all of this reduces to a clean FIFO ring with full capacity `N` — but the sequence-number scaffolding is exactly what lets it stay correct once the producers are real threads.
