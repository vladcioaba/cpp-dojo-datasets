## challenge: copy-on-write string, driven by use_count
tags: smart-pointers, raii
track: core
difficulty: hard

Implement `CowString`: copies are O(1) because they share one heap buffer through a `shared_ptr<std::string>`; the real copy happens only when someone *writes* to a shared buffer. `append` must detach — clone the buffer — if and only if other owners exist, and mutate in place when the string is the sole owner. The harness checks sharing (identical buffer addresses), correct detachment (writers never disturb other holders), and that a sole owner does not pay for a copy.

hint: `data_.use_count()` tells you how many CowStrings share the buffer. Greater than 1 at write time means detach first; exactly 1 means mutate in place.
hint: Detach = `data_ = std::make_shared<std::string>(*data_);` — clone the current contents into a fresh buffer, drop the old share. The OTHER holders keep the old buffer untouched; only the writer moves.
hint: The default copy constructor is already correct — copying the shared_ptr member shares the buffer and bumps use_count. All the cleverness lives in the write path.

```cpp
// starter
class CowString {
public:
    explicit CowString(std::string s)
        : data_(std::make_shared<std::string>(std::move(s))) {}

    // Copies share the buffer — that's the cheap part, and it already works.
    CowString(const CowString&) = default;
    CowString& operator=(const CowString&) = default;

    // Read access: never copies.
    const std::string& view() const { return *data_; }

    // TODO: write access. Mutating a SHARED buffer corrupts every other
    // holder. Detach (clone the buffer) first — but only when actually shared.
    void append(const std::string& tail) {
        *data_ += tail;
    }

    // How many CowStrings currently share this buffer.
    long owners() const { return data_.use_count(); }

private:
    std::shared_ptr<std::string> data_;
};
```

```cpp
class CowString {
public:
    explicit CowString(std::string s)
        : data_(std::make_shared<std::string>(std::move(s))) {}

    // Copies share the buffer — that's the cheap part, and it already works.
    CowString(const CowString&) = default;
    CowString& operator=(const CowString&) = default;

    // Read access: never copies.
    const std::string& view() const { return *data_; }

    // Write access: detach if shared, then mutate our (now private) buffer.
    void append(const std::string& tail) {
        detachIfShared();
        *data_ += tail;
    }

    // How many CowStrings currently share this buffer.
    long owners() const { return data_.use_count(); }

private:
    void detachIfShared() {
        if (data_.use_count() > 1) {
            data_ = std::make_shared<std::string>(*data_);  // clone, unshare
        }
    }

    std::shared_ptr<std::string> data_;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    CowString a(std::string("hello"));
    assert(a.owners() == 1);
    assert(a.view() == "hello");

    CowString b = a;                        // O(1) copy: share the buffer
    assert(a.owners() == 2);
    assert(b.owners() == 2);
    assert(&a.view() == &b.view());         // literally the same std::string

    b.append(" world");                     // write on shared -> b detaches
    assert(b.view() == "hello world");
    assert(a.view() == "hello");            // a is untouched
    assert(a.owners() == 1);
    assert(b.owners() == 1);
    assert(&a.view() != &b.view());

    const std::string* buf = &b.view();
    b.append("!");                          // sole owner -> in place, no clone
    assert(b.view() == "hello world!");
    assert(&b.view() == buf);               // same std::string object
    assert(b.owners() == 1);

    CowString c = b;
    CowString d = c;
    assert(b.owners() == 3);
    c.append("?");                          // only the WRITER detaches
    assert(c.owners() == 1);
    assert(b.owners() == 2);                // b and d still share
    assert(d.owners() == 2);
    assert(c.view() == "hello world!?");
    assert(b.view() == "hello world!");
    assert(d.view() == "hello world!");
    assert(&b.view() == &d.view());

    std::puts("PASS");
}
```

**Editorial:** Copy-on-write splits the cost of value semantics: reads and copies are shared-and-cheap, and the price of a real copy is deferred to the first write — paid by the writer, exactly once. `shared_ptr` supplies both ingredients: shared lifetime for the buffer, and `use_count()` as the sharing detector. The invariant to hold in your head: *a buffer may be referenced by many readers, but a mutation must only ever touch a buffer with use_count 1.* Hence `detachIfShared()` — clone when `use_count() > 1`, then mutate the private clone. The starter violates the invariant in the most instructive way: `b.append(" world")` writes through a buffer that `a` also holds, so `a` silently becomes `"hello world"` — spooky action at a distance, the same aliasing bug that made pre-C++11 COW `std::string` infamous. Note who moves on detach: the writer re-points to the clone; the other holders keep the *old* buffer at its old address, which the harness pins with address comparisons. Two honest caveats before using this in production: `use_count()` is only race-free here because the harness is single-threaded — under concurrency the check-then-write is a TOCTOU and the classic remedy is deep atomics or just not doing COW (which is why C++11 banned COW in `std::string`); and `owners()` leaking the count into the public API is for the exercise — real COW keeps detachment fully internal, triggered by the non-const access path.
