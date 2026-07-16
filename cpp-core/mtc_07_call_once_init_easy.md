## challenge: build it once — call_once
tags: concurrency, call-once, initialization
track: core
difficulty: easy

`Config` (provided by the harness) is expensive to build, and the harness counts every construction. Right now `ConfigCache::get()` builds a fresh one on *every* call — from four threads at once, no less. Make it lazy and thread-safe: exactly one `Config` is ever constructed, and every caller gets a reference to that same object. The members you need are already declared.

hint: The class already owns a `std::once_flag` — that is `std::call_once`'s bookmark. Pass it plus a callable; the callable runs exactly once no matter how many threads arrive.
hint: `std::call_once(flag_, [this] { cfg_ = std::make_unique<Config>(); });` — every other thread blocks until the winner finishes, then skips the callable.
hint: A double-checked `if (!cfg_)` without synchronization is the classic broken version — two threads can both see null and both construct. call_once (or a function-local static) is the correct tool.

```cpp
// starter
// Config is defined by the harness; each construction is counted.
// Make get() build the Config exactly once, no matter how many
// threads call it concurrently, and always return the same object.
class ConfigCache {
public:
    Config& get() {
        cfg_ = std::make_unique<Config>();   // BUG: builds one per call!
        return *cfg_;
    }
private:
    std::once_flag flag_;                    // use me
    std::unique_ptr<Config> cfg_;
};
```

```cpp
class ConfigCache {
public:
    Config& get() {
        std::call_once(flag_, [this] {       // runs exactly once, ever
            cfg_ = std::make_unique<Config>();
        });
        return *cfg_;                        // all callers: same object
    }
private:
    std::once_flag flag_;
    std::unique_ptr<Config> cfg_;
};
```

```cpp
// harness
#include <bits/stdc++.h>
std::atomic<int> g_built{0};
struct Config {
    int version = 7;
    Config() { g_built.fetch_add(1); }
};
//__USER__
int main() {
    ConfigCache cache;
    const Config* seen[4] = {nullptr, nullptr, nullptr, nullptr};
    std::vector<std::thread> threads;
    for (int t = 0; t < 4; ++t)
        threads.emplace_back([&cache, &seen, t] {
            for (int i = 0; i < 1000; ++i) {
                Config& c = cache.get();
                seen[t] = &c;
            }
        });
    for (auto& th : threads) th.join();
    assert(g_built.load() == 1);                       // built exactly once
    assert(seen[0] == seen[1] && seen[1] == seen[2]
                              && seen[2] == seen[3]);  // same object for all
    assert(cache.get().version == 7);
    std::puts("PASS");
}
```

**Editorial:** `std::call_once` + `std::once_flag` is the standard's "initialize exactly once" primitive: the first thread runs the callable while latecomers block, then everyone proceeds — and if the callable throws, the flag stays unset so the next caller retries. The starter builds 4,000 Configs (and races on the `unique_ptr` besides); the naive `if (!cfg_)` check narrows that to "sometimes two" — still wrong. Worth knowing the alternative: when the lazy object can live in a function-local `static`, C++11 magic statics give the same guarantee with less code (the Meyers singleton). `call_once` earns its keep when the target is a class member, as here, or when initialization needs runtime arguments.
