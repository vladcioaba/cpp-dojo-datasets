## challenge: a worker that schedules itself — and survives it
tags: smart-pointers, raii
track: core
difficulty: hard

`Worker::scheduleOn(q)` pushes a task that will call `doWork()` later — possibly after the caller has dropped every external `shared_ptr` to the worker. Each queued task must therefore co-own the worker, sharing the *original* control block (a fresh `std::shared_ptr(this)` would double-free). Fix the class so it can mint owning handles to itself. The harness drops the external handle before running the queue and checks the worker lives exactly as long as its pending tasks.

hint: The starter's lambda captures raw `this` — zero ownership, so the worker is destroyed the moment the caller's shared_ptr dies, and the queued task would call into a corpse. The harness catches it earlier: use_count never rises.
hint: `std::shared_ptr<Worker>(this)` is the trap, not the fix — the raw-pointer constructor always builds a NEW control block, giving the object two independent owner groups and two deletes.
hint: Inherit publicly from `std::enable_shared_from_this<Worker>` and capture `self = shared_from_this()` in the lambda. Requirement: the object must already be owned by a shared_ptr when you call it (the harness's make_shared satisfies that).

```cpp
// starter
class TaskQueue {
public:
    void push(std::function<void()> f) { tasks_.push_back(std::move(f)); }
    void runAll() {
        auto pending = std::move(tasks_);
        tasks_.clear();
        for (auto& f : pending) f();
    }   // pending (and the captures inside) destroyed here
private:
    std::vector<std::function<void()>> tasks_;
};

class Worker {
public:
    inline static int alive = 0;
    inline static int runs = 0;
    Worker() { ++alive; }
    ~Worker() { --alive; }

    // TODO: the queued task must CO-OWN this Worker so it stays alive until
    // the task has run — even if the caller drops every external handle.
    // Capturing raw `this` owns nothing. shared_ptr<Worker>(this) double-frees.
    void scheduleOn(TaskQueue& q) {
        q.push([this] { doWork(); });
    }

    void doWork() { ++runs; }
};
```

```cpp
class TaskQueue {
public:
    void push(std::function<void()> f) { tasks_.push_back(std::move(f)); }
    void runAll() {
        auto pending = std::move(tasks_);
        tasks_.clear();
        for (auto& f : pending) f();
    }   // pending (and the captures inside) destroyed here
private:
    std::vector<std::function<void()>> tasks_;
};

class Worker : public std::enable_shared_from_this<Worker> {
public:
    inline static int alive = 0;
    inline static int runs = 0;
    Worker() { ++alive; }
    ~Worker() { --alive; }

    void scheduleOn(TaskQueue& q) {
        // shared_from_this() joins the EXISTING owner group: the captured
        // `self` shares the control block make_shared created.
        q.push([self = shared_from_this()] { self->doWork(); });
    }

    void doWork() { ++runs; }
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    TaskQueue q;
    {
        auto w = std::make_shared<Worker>();
        assert(Worker::alive == 1);
        w->scheduleOn(q);
        w->scheduleOn(q);
        // w + one owning capture per queued task, all one control block.
        assert(w.use_count() == 3);
    }                               // caller drops its handle...
    assert(Worker::alive == 1);     // ...but the queued tasks keep it alive
    assert(Worker::runs == 0);

    q.runAll();                     // tasks execute, then captures destroyed
    assert(Worker::runs == 2);
    assert(Worker::alive == 0);     // last co-owner released the worker

    std::puts("PASS");
}
```

**Editorial:** Async code inverts lifetime: the scheduling call returns immediately, but the object must survive until an unknowable later moment when the task runs. The only honest answer is for the task itself to own the object. The starter's `[this]` capture owns nothing — after the caller's brace, the worker is destroyed and the queued call would be UB; the harness surfaces it deterministically as `use_count() == 1` instead of 3, failing before anything dangles. The reflex fix, `std::shared_ptr<Worker>(this)`, is the famous double-free: that constructor unconditionally mints a *new* control block, so the worker now has two owner groups of count 1, each of which will `delete` it. `enable_shared_from_this` exists precisely for this: the base holds a hidden `weak_ptr` that the first `shared_ptr` (here, `make_shared`) initializes, and `shared_from_this()` locks it — returning a handle in the original owner group, count bumped from 1 to 2 to 3 as the harness observes. Preconditions matter: the base must be inherited *publicly*, and the object must already be shared-owned when `shared_from_this()` runs — on a stack object or inside the constructor it throws `std::bad_weak_ptr`. Watch the endgame too: `runAll` moves the task vector out, runs it, and the captures die with `pending` — the worker's destruction point is exactly the last task's destruction, which is the contract async frameworks (ASIO's `self = shared_from_this()` idiom) are built on.
