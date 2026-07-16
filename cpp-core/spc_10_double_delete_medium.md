## challenge: fix: two owners, one session, one crash
tags: smart-pointers, raii
track: core
difficulty: medium

This code review found a bug: a service that hands out a session through two `shared_ptr` handles crashes at shutdown with a double free — heap-corruption reports point at the session's memory. The counting looks off well before the crash, too: with both handles live, each reports `use_count() == 1`. Find and fix it — `makeHandles` must keep returning two handles to one shared session.

hint: Compare the use_counts the harness expects with what the starter produces. Two owners of one object should both report 2 — why would each say 1?
hint: `std::shared_ptr<T>(raw)` doesn't look up whether `raw` is already owned — it can't. Every construction from a raw pointer creates a brand-new control block that believes it is the sole owner.
hint: Only ONE shared_ptr may ever be built from a given raw pointer; every other owner must be a copy of it. Better: never touch the raw pointer — `make_shared` once, copy for the second handle.

```cpp
// starter
struct Session {
    inline static int alive = 0;
    int id = 42;
    Session() { ++alive; }
    ~Session() { --alive; }
};

// Returns two owning handles to ONE shared session.
std::pair<std::shared_ptr<Session>, std::shared_ptr<Session>> makeHandles() {
    Session* raw = new Session();
    std::shared_ptr<Session> primary(raw);
    std::shared_ptr<Session> secondary(raw);    // second owner of same raw ptr
    return {primary, secondary};
}
```

```cpp
struct Session {
    inline static int alive = 0;
    int id = 42;
    Session() { ++alive; }
    ~Session() { --alive; }
};

// Returns two owning handles to ONE shared session.
std::pair<std::shared_ptr<Session>, std::shared_ptr<Session>> makeHandles() {
    auto primary = std::make_shared<Session>();
    auto secondary = primary;       // copy: same control block, count == 2
    return {primary, secondary};
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    {
        auto [a, b] = makeHandles();
        assert(a != nullptr && b != nullptr);
        assert(a.get() == b.get());     // one session...
        assert(a->id == 42);

        // ...with ONE owner group. Two control blocks would each say 1 here,
        // and each would delete the session on its way out.
        assert(a.use_count() == 2);
        assert(b.use_count() == 2);
        assert(Session::alive == 1);
    }
    assert(Session::alive == 0);        // destroyed exactly once
    std::puts("PASS");
}
```

**Editorial:** The strong count lives in the control block, and the control block is created by the `shared_ptr` constructor — `std::shared_ptr<Session>(raw)` cannot discover that `raw` is already owned, so the starter builds *two* control blocks over one object, each with count 1. Both handles work fine all day; the corruption is scheduled for later, when the second control block drains and calls `delete` on already-freed memory. The tell is exactly what the harness asserts: two live owners of one session, yet `use_count() == 1` on each — whenever `use_count()` disagrees with the number of owners you can see, suspect a second control block (checking this *before* scope exit is also what lets the harness fail cleanly instead of crashing in the double delete). The fix restates the rule: an object enters `shared_ptr` management exactly once, and every additional owner is a *copy* of an existing handle. `make_shared` makes the rule structural — no raw pointer ever exists to be wrapped twice, and you get the fused single allocation for free. The same landmine has a second trigger worth knowing: passing `ptr.get()` into an API that wraps it in its own `shared_ptr` — and its principled cousin, `shared_ptr(this)`, is why `enable_shared_from_this` exists.
