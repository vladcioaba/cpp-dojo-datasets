## challenge: read-mostly phone book
tags: concurrency, shared-mutex, readers-writers
track: core
difficulty: medium

Wrap a `std::map` so many threads can `lookup()` simultaneously while `update()` gets exclusive access — the readers-writers pattern with `std::shared_mutex`. The harness hammers it with three reader threads while the main thread performs 5,000 updates. Readers must never block each other; a plain `std::mutex` would work but wastes the read parallelism this exercise is about.

hint: Two lock modes, two RAII types: writers take `std::unique_lock<std::shared_mutex>`, readers take `std::shared_lock<std::shared_mutex>`.
hint: `lookup()` is const but still locks — declare the shared_mutex `mutable`. Return `std::nullopt` when the key is missing rather than throwing.
hint: `map::find` under the shared lock, `map::operator[]` (or insert_or_assign) under the unique lock. Copy the value out before the lock is released — returning a reference into the map would dangle past the lock.

```cpp
// starter
// Many concurrent readers, occasional writer. Readers must be able
// to run in parallel with each other: use std::shared_mutex.
class PhoneBook {
public:
    void update(const std::string& name, int number) {
        // TODO: exclusive lock, then write to entries_
    }
    std::optional<int> lookup(const std::string& name) const {
        // TODO: shared lock, then read from entries_
        return std::nullopt;
    }
private:
    // TODO: a mutable std::shared_mutex member
    std::map<std::string, int> entries_;
};
```

```cpp
class PhoneBook {
public:
    void update(const std::string& name, int number) {
        std::unique_lock<std::shared_mutex> lock(m_);  // exclusive: one writer
        entries_[name] = number;
    }
    std::optional<int> lookup(const std::string& name) const {
        std::shared_lock<std::shared_mutex> lock(m_);  // shared: readers overlap
        auto it = entries_.find(name);
        if (it == entries_.end()) return std::nullopt;
        return it->second;                             // copied out under lock
    }
private:
    mutable std::shared_mutex m_;   // mutable: const readers still lock
    std::map<std::string, int> entries_;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    PhoneBook book;
    book.update("alice", 100);
    book.update("bob", 200);
    assert(book.lookup("alice") == 100);        // basic write-then-read
    assert(book.lookup("bob") == 200);
    assert(!book.lookup("nobody").has_value()); // missing key

    std::atomic<bool> ok{true};
    std::vector<std::thread> readers;
    for (int t = 0; t < 3; ++t)
        readers.emplace_back([&] {
            for (int i = 0; i < 20000; ++i) {
                auto a = book.lookup("alice");           // stable entry
                if (a != 100) ok = false;
                auto c = book.lookup("carol");           // entry in flux
                if (c && (*c < 1 || *c > 5000)) ok = false;
            }
        });
    for (int i = 1; i <= 5000; ++i) book.update("carol", i);  // writer
    for (auto& r : readers) r.join();
    assert(ok.load());                          // no torn or bogus reads
    assert(book.lookup("carol") == 5000);       // last write wins
    std::puts("PASS");
}
```

**Editorial:** `std::shared_mutex` has two modes — any number of shared holders, or one exclusive holder — mapped onto RAII by `std::shared_lock` and `std::unique_lock`. The visibility guarantees match a plain mutex; this is purely a throughput play for read-mostly data. The idiomatic details carry the exercise: the mutex is `mutable` so `const` readers can lock (const = reader is exactly the convention shared_mutex rewards), and `lookup` returns the value *by copy* inside an `optional` — returning `const int&` into the map would dangle the instant the lock released and the writer touched the tree. Caveats worth repeating in review: shared locking costs more per acquisition than a plain mutex, and writers can starve under heavy read traffic — measure before committing.
