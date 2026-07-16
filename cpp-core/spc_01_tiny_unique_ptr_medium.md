## challenge: build a tiny UniquePtr from scratch
tags: smart-pointers, raii
track: core
difficulty: medium

Implement `UniquePtr<T>`: sole ownership of a heap object, destroyed exactly once, transferable only by move. Fill in the destructor, the move operations, `release()` and `reset()`, and make copying impossible. The harness counts constructions and destructions of a probe type — every leak, double delete, or accidental copy shows up as a wrong count.

hint: The move constructor must steal `other.ptr_` AND leave `other.ptr_` null — otherwise two objects both delete it.
hint: Move assignment has to destroy whatever `*this` currently owns before stealing; guard against self-assignment. `release()` gives the pointer up without deleting; `reset()` deletes then takes.
hint: Declaring move operations already suppresses the copies, but write `UniquePtr(const UniquePtr&) = delete;` (and same for copy assignment) explicitly — ownership types should say it out loud. `delete` on nullptr is a safe no-op, so the destructor needs no if.

```cpp
// starter
template <typename T>
class UniquePtr {
public:
    UniquePtr() = default;
    explicit UniquePtr(T* p) : ptr_(p) {}

    // TODO: destructor — destroy the owned object (delete on nullptr is fine).
    ~UniquePtr() {}

    // TODO: forbid copy construction and copy assignment explicitly.

    // TODO: move constructor — steal other's pointer, leave other empty.
    UniquePtr(UniquePtr&& other) noexcept {}

    // TODO: move assignment — destroy what we own, then steal other's pointer.
    UniquePtr& operator=(UniquePtr&& other) noexcept { return *this; }

    T* get() const { return ptr_; }
    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }

    // TODO: release() — give up ownership; return the raw pointer, own nothing.
    T* release() { return nullptr; }

    // TODO: reset(p) — destroy what we own, then own p.
    void reset(T* p = nullptr) {}

private:
    T* ptr_ = nullptr;
};
```

```cpp
template <typename T>
class UniquePtr {
public:
    UniquePtr() = default;
    explicit UniquePtr(T* p) : ptr_(p) {}

    ~UniquePtr() { delete ptr_; }

    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;

    UniquePtr(UniquePtr&& other) noexcept : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }

    UniquePtr& operator=(UniquePtr&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }

    T* get() const { return ptr_; }
    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }

    T* release() {
        T* p = ptr_;
        ptr_ = nullptr;
        return p;
    }

    void reset(T* p = nullptr) {
        delete ptr_;
        ptr_ = p;
    }

private:
    T* ptr_ = nullptr;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
struct Probe {
    inline static int alive = 0;
    inline static int destroyed = 0;
    int v;
    explicit Probe(int x) : v(x) { ++alive; }
    ~Probe() { ++destroyed; --alive; }
};

int main() {
    {
        UniquePtr<Probe> p(new Probe(7));
        assert(p.get() != nullptr);
        assert((*p).v == 7);
        assert(p->v == 7);
        assert(Probe::alive == 1);

        UniquePtr<Probe> q(std::move(p));   // move ctor: steal, no copy
        assert(p.get() == nullptr);
        assert(q->v == 7);
        assert(Probe::alive == 1);

        UniquePtr<Probe> r(new Probe(9));
        assert(Probe::alive == 2);
        r = std::move(q);                   // move assign: old 9 destroyed
        assert(Probe::alive == 1);
        assert(Probe::destroyed == 1);
        assert(q.get() == nullptr);
        assert(r->v == 7);

        Probe* raw = r.release();           // ownership given up, no delete
        assert(r.get() == nullptr);
        assert(Probe::alive == 1);
        delete raw;
        assert(Probe::alive == 0);

        r.reset(new Probe(3));
        assert(r->v == 3);
        assert(Probe::alive == 1);
        r.reset();                          // reset() destroys
        assert(Probe::alive == 0);
    }
    assert(Probe::alive == 0);
    assert(Probe::destroyed == 3);          // 7, 9, 3 — each exactly once
    static_assert(!std::is_copy_constructible_v<UniquePtr<Probe>>,
                  "UniquePtr must not be copyable");
    static_assert(!std::is_copy_assignable_v<UniquePtr<Probe>>,
                  "UniquePtr must not be copy-assignable");
    std::puts("PASS");
}
```

**Editorial:** The whole exercise is the ownership invariant: at any moment, exactly one `UniquePtr` believes it owns the pointer. Every member is a small proof obligation against it. The move constructor must null out the source — copying the pointer without nulling leaves two owners and a double delete at scope exit. Move assignment must first `delete ptr_` (else the old object leaks) and guard `this != &other` (else self-move deletes the object it is about to steal). `release()` is the deliberate hole in RAII — ownership exits the type system, so the harness has to `delete` manually — while `reset()` re-enters it. Note the asymmetry: `release()` never deletes, `reset()` always does. Declaring the move operations already makes the class non-copyable (implicit copies are suppressed), but real ownership types spell out `= delete` so the intent survives refactoring. This is, minus the deleter parameter and a few conveniences, exactly `std::unique_ptr` — pointer-sized, zero overhead, and the reason manual `delete` disappeared from modern C++.
