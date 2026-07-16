## challenge: pImpl with unique_ptr
tags: smart-pointers, raii
track: core
difficulty: medium

`Tracker` hides its state behind the pImpl idiom: the class body holds only `std::unique_ptr<Impl>` with `Impl` merely forward-declared, and every member is defined out of line, after `Impl` is complete. Implement `Impl` (running sum and sample count) and the member functions — including working move operations. The harness checks the arithmetic, that moves carry the state over, and that the class is (correctly) non-copyable.

hint: `Impl` needs the sum and the number of samples; the constructor should `std::make_unique<Impl>()`.
hint: The destructor and move operations can all be `= default` — but only in a spot where `Impl` is a complete type. That placement is the entire pImpl trick: `unique_ptr<Impl>`'s deleter instantiates there.
hint: Defaulted move operations on a `unique_ptr` member do the right thing: the impl pointer transfers, the source is left empty, and move-assign releases the target's old Impl.

```cpp
// starter
// Public interface: no data members visible except one opaque pointer.
class Tracker {
public:
    Tracker();
    ~Tracker();
    Tracker(Tracker&& other) noexcept;
    Tracker& operator=(Tracker&& other) noexcept;

    void add(int x);        // record a sample
    int total() const;      // sum of all samples
    int count() const;      // number of samples

private:
    struct Impl;            // incomplete here — that's the point
    std::unique_ptr<Impl> impl_;
};

// TODO: define Tracker::Impl with the state (sum, sample count).
struct Tracker::Impl { /* TODO */ };

Tracker::Tracker() : impl_(std::make_unique<Impl>()) {}
Tracker::~Tracker() = default;

// TODO: make the move operations actually move the impl over.
Tracker::Tracker(Tracker&& other) noexcept : impl_(nullptr) {}
Tracker& Tracker::operator=(Tracker&& other) noexcept { return *this; }

// TODO: implement through impl_.
void Tracker::add(int x) {}
int Tracker::total() const { return 0; }
int Tracker::count() const { return 0; }
```

```cpp
// Public interface: no data members visible except one opaque pointer.
class Tracker {
public:
    Tracker();
    ~Tracker();
    Tracker(Tracker&& other) noexcept;
    Tracker& operator=(Tracker&& other) noexcept;

    void add(int x);        // record a sample
    int total() const;      // sum of all samples
    int count() const;      // number of samples

private:
    struct Impl;            // incomplete here — that's the point
    std::unique_ptr<Impl> impl_;
};

struct Tracker::Impl {
    int sum = 0;
    int samples = 0;
};

// All special members defined AFTER Impl is complete: unique_ptr's deleter
// is instantiated here, where it can see ~Impl.
Tracker::Tracker() : impl_(std::make_unique<Impl>()) {}
Tracker::~Tracker() = default;
Tracker::Tracker(Tracker&& other) noexcept = default;
Tracker& Tracker::operator=(Tracker&& other) noexcept = default;

void Tracker::add(int x) {
    impl_->sum += x;
    ++impl_->samples;
}
int Tracker::total() const { return impl_->sum; }
int Tracker::count() const { return impl_->samples; }
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Tracker t;
    t.add(5);
    t.add(7);
    assert(t.total() == 12);
    assert(t.count() == 2);

    Tracker u = std::move(t);       // move ctor: impl pointer transfers
    u.add(3);
    assert(u.total() == 15);
    assert(u.count() == 3);

    Tracker v;
    v.add(100);
    v = std::move(u);               // move assign: v's old impl released
    assert(v.total() == 15);
    assert(v.count() == 3);

    // unique_ptr member => moves only, no copies. That is the correct
    // default for a pImpl class.
    static_assert(!std::is_copy_constructible_v<Tracker>,
                  "pImpl with unique_ptr is move-only");
    static_assert(!std::is_copy_assignable_v<Tracker>,
                  "pImpl with unique_ptr is move-only");
    static_assert(std::is_nothrow_move_constructible_v<Tracker>);
    std::puts("PASS");
}
```

**Editorial:** pImpl trades one indirection for a compile-time firewall: the header exposes nothing but a forward declaration and a `unique_ptr`, so `Impl` can change freely without recompiling users. The load-bearing subtlety is *where* the special members are defined. `std::unique_ptr<Impl>` happily exists with an incomplete `Impl` — but destroying (or move-assigning) it instantiates the deleter, which calls `delete` on an `Impl*` and therefore demands the complete type. If you let the compiler generate `~Tracker` implicitly in the class body, that instantiation happens in the header, where `Impl` is still incomplete — a hard error (or worse, in sloppier setups, UB from deleting an incomplete type). Hence the ritual: declare `~Tracker();` in the class, write `Tracker::~Tracker() = default;` after `Impl`'s definition. Same for both move operations. Once placed correctly, `= default` does everything: moving a `Tracker` just moves the impl pointer (nothrow, pointer-cheap), and move-assignment automatically releases the target's old `Impl`. Copyability is gone because `unique_ptr` is move-only — the right default; if deep copies are wanted, you write them deliberately by cloning the `Impl`.
