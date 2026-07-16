## challenge: a worker that stops when asked
tags: concurrency, jthread, cancellation
track: core
difficulty: medium

Implement `pollSensor`, a worker loop for a `std::jthread`. It bumps the sample counter and yields, over and over — until stop is requested, at which point it must return promptly. The harness lets it run, then destroys the jthread: the destructor issues `request_stop()` and joins, and the counter must stand still afterwards. This is C++20 cooperative cancellation — no hand-rolled `atomic<bool> quit` flag.

hint: When a jthread's callable takes `std::stop_token` as its first parameter, the jthread supplies one automatically, connected to its internal stop_source.
hint: The loop shape is: `while (!st.stop_requested()) { ...work...; std::this_thread::yield(); }` — poll the token once per iteration and simply return when it fires.
hint: "Cooperative" means the thread exits at its next check — nothing is killed mid-operation. The jthread destructor then joins cleanly. If your loop never consults the token, the destructor waits forever.

```cpp
// starter
// Worker for a std::jthread: count samples until asked to stop,
// then return promptly. The jthread passes the stop_token itself.
void pollSensor(std::stop_token st, std::atomic<long>& samples) {
    // TODO: while stop has NOT been requested:
    //         samples.fetch_add(1); std::this_thread::yield();
}
```

```cpp
void pollSensor(std::stop_token st, std::atomic<long>& samples) {
    while (!st.stop_requested()) {                    // cooperative check
        samples.fetch_add(1, std::memory_order_relaxed);
        std::this_thread::yield();                    // be a polite spinner
    }
}   // returns at the first check after request_stop()
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::atomic<long> samples{0};
    {
        std::jthread worker(pollSensor, std::ref(samples));
        // give the worker a moment to prove it is alive
        for (int i = 0; i < 500000 && samples.load() < 1000; ++i)
            std::this_thread::yield();
        assert(samples.load() >= 1000);      // it ran
    }   // ~jthread: request_stop() then join() — must return promptly
    long frozen = samples.load();
    std::this_thread::sleep_for(std::chrono::milliseconds(50));
    assert(samples.load() == frozen);        // fully stopped: counter is still
    std::puts("PASS");
}
```

**Editorial:** `std::jthread` fixes the two chronic `std::thread` diseases at once: its destructor calls `request_stop()` and then `join()` (no more `std::terminate` from a forgotten join), and it carries a built-in cancellation channel — a `std::stop_source` whose `std::stop_token` is handed to your callable when the first parameter slot asks for it. Cancellation is *cooperative*: `request_stop()` just flips a flag; the worker exits at its next `stop_requested()` check, finishing its current iteration and unwinding normally — contrast `pthread_cancel`, which can kill a thread while it holds a mutex. The harness checks both halves: the worker made progress, and after the destructor it made none. For workers that sleep on a condition variable instead of looping, `condition_variable_any::wait(lock, stop_token, pred)` is the stop-aware wait.
