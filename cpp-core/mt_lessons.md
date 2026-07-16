# Multithreading

10 lessons that build from "what is a thread" to "why your lock-free queue is broken" — the concurrency track every C++ interview loops back to.

## fact: Threads share everything — that is the power and the problem
tags: concurrency, threads
track: core

A process owns an address space; a thread is just an execution path inside it. Two processes must be tricked into sharing memory (pipes, sockets, shm); two threads cannot be stopped from sharing it — every global, every heap object, every pointer you pass is visible to all of them. That is why threads are cheap to communicate with and easy to corrupt.

`std::thread` starts running its callable immediately on construction — there is no `start()`. Before the `std::thread` object is destroyed you must decide its fate: `join()` blocks until the thread finishes, `detach()` abandons it to run on its own. Destroying a joinable thread calls `std::terminate` — the standard refuses to guess. Treat `detach()` with suspicion: a detached thread that touches objects owned by someone else is a use-after-free waiting for a slow day. In practice, almost every thread you write should be joined, and C++20's `std::jthread` (lesson 7) automates exactly that.

Pass arguments by value by default; `std::ref` when you genuinely want to share — and then you have signed up for the synchronization lessons that follow.

```cpp
#include <thread>
#include <cstdio>

int main() {
    int result = 0;
    std::thread worker([&result] {      // starts immediately
        result = 6 * 7;                  // shares main's stack variable
    });
    worker.join();                       // wait — mandatory before ~thread
    std::printf("%d\n", result);         // 42, safe: join synchronizes
}
```

## fact: Data races are UB — a mutex makes them impossible
tags: concurrency, mutex
track: core

A data race is precise: two threads touch the same memory location, at least one is a write, and nothing orders the accesses. The standard's penalty is not "a stale value" — it is undefined behavior. `++count` compiles to read, add, write; two threads interleaving those steps lose increments, and the optimizer is allowed to assume it never happens.

`std::mutex` fixes this by mutual exclusion: only one thread at a time may hold it, and everything done inside the critical section becomes visible to the next thread that acquires it. But never call `lock()`/`unlock()` by hand — an early return or exception between them leaves the mutex locked forever. Recall RAII: `std::lock_guard` locks in its constructor and unlocks in its destructor, on every exit path. C++17's `std::scoped_lock` is the same idea and also takes *several* mutexes at once, locking them with a deadlock-avoidance algorithm (lesson 10 shows why that matters).

The discipline: associate each mutex with the exact data it guards — same class, adjacent declaration — and hold it for every read *and* write of that data. A mutex you only take when writing protects nothing.

```cpp
class SafeCounter {
public:
    void increment() {
        std::lock_guard<std::mutex> lock(m_);  // unlocks on any exit
        ++count_;
    }
    long value() const {
        std::lock_guard<std::mutex> lock(m_);  // readers lock too!
        return count_;
    }
private:
    mutable std::mutex m_;   // mutable: lockable in const methods
    long count_ = 0;
};
```

## fact: condition_variable — sleep until something is true
tags: concurrency, condition-variable
track: core

Polling a flag in a loop burns CPU; a `std::condition_variable` lets a thread sleep until another thread announces a change. The waiting side takes a `std::unique_lock` (not `lock_guard` — the wait must be able to unlock and relock, so it needs the movable, lock-again-later kind), then calls `wait`. Atomically, `wait` releases the mutex and puts the thread to sleep; when notified, it relocks before returning. The notifying side changes the shared state under the same mutex, then calls `notify_one()` or `notify_all()`.

The rule that separates working code from interview failure: **always wait in a predicate loop**. The OS may wake a waiter with no notification at all — a *spurious wakeup* — and even a real notification proves only that the condition *was* true; another thread may have consumed it before you relocked. `cv.wait(lock, pred)` is exactly that loop: `while (!pred()) wait(lock);`. Write the condition, not the handshake.

Second rule: the predicate's data must be modified under the mutex, or a waiter can check the predicate, decide to sleep, and miss the notification in the gap — the *lost wakeup*.

```cpp
std::mutex m;
std::condition_variable cv;
std::deque<int> inbox;

void consumer() {
    std::unique_lock<std::mutex> lock(m);
    cv.wait(lock, [] { return !inbox.empty(); }); // predicate loop
    int job = inbox.front(); inbox.pop_front();
    // lock still held here
}
void producer(int job) {
    { std::lock_guard<std::mutex> lock(m); inbox.push_back(job); }
    cv.notify_one();
}
```

## fact: std::atomic — when a mutex is overkill
tags: concurrency, atomics
track: core

If the shared state is a single counter, flag, or pointer, a mutex is a sledgehammer: kernel arbitration, cache-line ping-pong on the lock itself, and a blocked thread on contention. `std::atomic<T>` makes individual loads, stores, and read-modify-write operations (`fetch_add`, `exchange`, `compare_exchange_strong`) indivisible — usually a single lock-free CPU instruction (`is_lock_free()` tells you).

`counter.fetch_add(1)` is the atomic answer to the lost-increment race from lesson 2: the read-add-write happens as one step, no interleaving possible. `std::atomic<bool> done` is the idiomatic "please stop" flag between threads — a plain `bool` there is a data race and genuinely miscompiles: the optimizer may hoist the load out of the loop and spin forever.

Know the boundary, because interviewers probe it. Atomics protect *one object per operation*. The moment an invariant spans two variables — balance and audit log, head and tail, "check then act" — separate atomic operations can interleave and break it. `if (count.load() > 0) count.fetch_add(-1);` is not atomic as a whole. That is mutex territory, or a single `compare_exchange` loop. Rule of thumb: atomics for independent scalars and flags, mutexes for invariants.

```cpp
std::atomic<long> hits{0};
std::atomic<bool> stop{false};

void worker() {
    while (!stop.load()) {
        hits.fetch_add(1);           // never loses an increment
    }
}
// elsewhere: stop.store(true);      // all workers exit promptly
```

## fact: Memory orderings — relaxed, acquire, release, and happens-before
tags: concurrency, memory-model
track: core

Every `std::atomic` operation takes a memory order, and this is the interview classic. Default is `memory_order_seq_cst`: all seq_cst operations across all threads appear in one global order — easiest to reason about, occasionally slower.

`memory_order_relaxed` guarantees only atomicity: the operation itself is indivisible, but it orders *nothing else*. Perfect for a statistics counter no one reads until the threads join; disastrous for a "data is ready" flag.

The pair that matters: a **release** store publishes, an **acquire** load subscribes. If thread B's acquire load reads the value written by thread A's release store, then everything A wrote *before* the release is visible to B *after* the acquire. That edge is called *happens-before*, and it is the entire game: a data race is exactly two conflicting accesses with no happens-before between them. Mutexes give you the same edge implicitly — unlock releases, lock acquires — which is why lesson 2 "just worked."

The classic exam question: with both operations relaxed below, may the assert fire? Yes — without the release/acquire pair, nothing stops the reader from seeing `ready == true` yet a stale `payload`. Store the data, *then* release the flag; acquire the flag, *then* read the data.

```cpp
int payload = 0;
std::atomic<bool> ready{false};

void writer() {
    payload = 42;                                    // A
    ready.store(true, std::memory_order_release);    // B: publish A
}
void reader() {
    while (!ready.load(std::memory_order_acquire)) {} // C: sees B...
    assert(payload == 42);                            // ...so A is visible
}
```

## fact: async, future, promise, packaged_task — getting a value back out
tags: concurrency, futures
track: core

`std::thread` returns nothing — smuggling a result out through a captured reference is the manual, error-prone way. The futures machinery is the typed channel: a `std::future<T>` is a one-shot handle to a value (or exception) that will exist later; `get()` blocks for it, and may be called once.

Three producers of futures, from highest level to lowest. `std::async(std::launch::async, f, args...)` runs `f` on another thread and returns its result as a future — exceptions thrown inside travel through and rethrow at `get()`. Gotcha the interviewer knows: with the default policy, the system may pick `deferred`, meaning `f` runs lazily on the *calling* thread at `get()` — pass `std::launch::async` explicitly when you mean concurrency. Second gotcha: a future from `std::async` blocks in its destructor until the task finishes — ignoring the return value serializes your "parallel" code.

`std::packaged_task<R(Args...)>` wraps a callable and exposes a future, but *you* choose where it runs — the building block for thread pools (lesson 9). `std::promise<T>` is the raw pipe: you `set_value` manually from anywhere. One writer, one reader, once.

```cpp
long long sum(const std::vector<int>& v, std::size_t lo, std::size_t hi);

long long parallel_sum(const std::vector<int>& v) {
    auto half = v.size() / 2;
    auto lower = std::async(std::launch::async,     // really a new thread
                            sum, std::cref(v), 0, half);
    long long upper = sum(v, half, v.size());       // this thread does the rest
    return lower.get() + upper;                     // blocks, rethrows if needed
}
```

## fact: jthread and stop_token — C++20 threads that clean up after themselves
tags: concurrency, jthread
track: core

Two chronic `std::thread` diseases: forgetting to `join()` (instant `std::terminate` when the object dies joinable) and having no standard way to *ask* a thread to stop — everyone hand-rolls an `atomic<bool>` flag. C++20's `std::jthread` cures both.

Its destructor calls `request_stop()` and then `join()` — a thread object you can simply let go out of scope, RAII applied to execution itself. Early returns and exceptions in the owning scope now clean up the thread instead of killing the process.

Cooperative cancellation is built in. If the callable's first parameter is a `std::stop_token`, the jthread passes one automatically, wired to its internal `std::stop_source`. The loop polls `st.stop_requested()` and exits at the next safe point — nothing is killed mid-instruction; the thread finishes its current iteration, releases its resources, unwinds normally. That "cooperative" is the point: preemptive thread killing (looking at you, `pthread_cancel`) can drop a mutex locked forever.

For threads blocked on a condition variable rather than looping, `std::condition_variable_any::wait(lock, stop_token, pred)` wakes when stop is requested, and `std::stop_callback` runs arbitrary code on request — the escape hatches that make cancellation reach sleeping threads.

```cpp
void sample_sensors();
void do_other_work();

void poller(std::stop_token st) {
    while (!st.stop_requested()) {        // cheap check, each iteration
        sample_sensors();
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}   // exits cleanly at the next check

int main() {
    std::jthread t(poller);   // stop_token supplied automatically
    do_other_work();
}   // ~jthread: request_stop() + join() — no terminate, no leak
```

## fact: shared_mutex — many readers or one writer
tags: concurrency, shared-mutex
track: core

A plain mutex serializes readers that never conflict with each other. For read-mostly data — config lookups, routing tables, caches read a million times per update — that is pure waste. `std::shared_mutex` (C++17) has two modes: any number of threads may hold it *shared*, or exactly one may hold it *exclusive*, never both.

The RAII pairing: readers take `std::shared_lock<std::shared_mutex>`, writers take `std::unique_lock<std::shared_mutex>` (or `lock_guard`/`scoped_lock`). Same acquire/release visibility guarantees as a plain mutex — this is a throughput optimization, not a different correctness model. The reader path pairs naturally with `const` methods and a `mutable` mutex, so the type system helps police who counts as a reader.

Honest caveats, because interviewers ask. Shared locking is *more* expensive per acquisition than a plain mutex — the win only appears when read critical sections are long or readers are many; benchmark before reaching for it. Under a constant stream of readers, a writer may starve (the standard leaves the fairness policy to the implementation). And a reader that mutates anything — even a cached field — under a shared lock is a data race with the other readers; that is what `mutable std::mutex` on the side or an upgrade-to-exclusive redesign is for.

```cpp
class Directory {
public:
    std::optional<int> find(const std::string& k) const {
        std::shared_lock lock(m_);          // readers run in parallel
        auto it = map_.find(k);
        return it == map_.end() ? std::nullopt : std::optional{it->second};
    }
    void set(const std::string& k, int v) {
        std::unique_lock lock(m_);          // writer waits them all out
        map_[k] = v;
    }
private:
    mutable std::shared_mutex m_;
    std::map<std::string, int> map_;
};
```

## fact: Thread pools — stop paying thread startup costs
tags: concurrency, thread-pool
track: core

Spawning a thread costs a system call, kernel bookkeeping, and a fresh stack (megabytes of address space) — tens of microseconds before your task runs a single instruction. Spawn one per request and a busy server drowns in threads: more runnable threads than cores means the scheduler burns time context-switching instead of computing (*oversubscription*).

The fix is the thread pool, and it is just lessons 2, 3, and 6 assembled: N worker threads created once (N ≈ `std::thread::hardware_concurrency()`), a queue of `std::function<void()>` or `std::packaged_task` guarded by a mutex, and a condition variable so idle workers sleep instead of spin. `submit()` pushes a task and notifies; each worker loops pop-task, run-task. Results come back through the futures machinery — wrap the callable in a `packaged_task`, hand its future to the caller, push the task into the queue.

Shutdown is the part people get wrong: set a `stopping` flag *under the mutex*, `notify_all()`, and make workers wait on the predicate "queue non-empty **or** stopping" — a worker sleeping on "non-empty" alone never wakes up to be told goodbye. C++ still has no standard pool (executors keep slipping), so every codebase has one; be the person who can whiteboard it.

```cpp
// The heart of every pool — the worker loop:
struct ThreadPool {
    std::mutex m_;
    std::condition_variable cv_;
    std::deque<std::function<void()>> queue_;
    bool stopping_ = false;

    void worker_loop() {
        for (;;) {
            std::function<void()> task;
            {
                std::unique_lock<std::mutex> lock(m_);
                cv_.wait(lock, [&] { return stopping_ || !queue_.empty(); });
                if (stopping_ && queue_.empty()) return;   // drained: exit
                task = std::move(queue_.front());
                queue_.pop_front();
            }       // unlock BEFORE running — never run user code under the lock
            task();
        }
    }
};
```

## fact: The concurrency bugs everyone writes once
tags: concurrency, debugging
track: core

You now have all the tools; here is how they cut people. Four classics to check for in every review — including your own.

**ABBA deadlock.** Thread 1 locks A then B; thread 2 locks B then A; each holds one and waits forever for the other. Fix: one global lock order everywhere (document it, or order by address), or take both in a single `std::scoped_lock(a, b)`, which internally uses the `std::lock` deadlock-avoidance algorithm. Never call a callback or unknown virtual while holding a lock — the "other" lock in an ABBA is usually hiding inside someone else's code.

**Forgotten join.** Any path that destroys a joinable `std::thread` — an early return, a thrown exception — is `std::terminate`. Use `std::jthread` and the bug class disappears.

**Capturing the loop variable by reference.** `for (int i = 0; i < 4; ++i) threads.emplace_back([&] { work(i); });` — every thread reads `i` *later*, mid-mutation or dangling after the loop. Capture per-thread values by copy: `[i]`. `[&]` on anything that outlives or races the loop body is the sharpest edge in the language.

**Locking only the writes.** Readers of guarded data need the same mutex (lesson 2) — a read-side race is still UB, even if it "only" reads.

```cpp
void process_chunk(int i);

void run_all() {
    std::vector<std::jthread> pool;
    for (int i = 0; i < 4; ++i) {
        pool.emplace_back([i] {          // [i] by copy — NOT [&]
            process_chunk(i);
        });
    }
}   // jthreads join themselves; no terminate, no dangling i
```
