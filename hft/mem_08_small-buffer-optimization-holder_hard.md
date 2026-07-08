## challenge: Small-buffer-optimization holder
tags: small-buffer-optimization, placement-new, type-erasure
track: hft
difficulty: hard

Heap allocation on the hot path is death by a thousand `malloc`s. A small-buffer-optimization (SBO) holder stores an object *inline* in a fixed aligned buffer — no heap — as long as it fits. Implement `SboHolder<Cap, Align>`: `emplace<T>(args...)` constructs a `T` in the inline buffer via placement `new` and returns a `T*`; `reset()` destroys the current object; `has_value()` reports occupancy; the destructor cleans up. Support holding *any* type that fits — including types whose destructor must run — by remembering how to destroy the current object.

Constraints: storage is a fixed inline `alignas(Align) unsigned char[Cap]` — no dynamic allocation ever. Reject a `T` too large or too over-aligned at compile time with `static_assert`. `emplace` over an existing value destroys the old one first; the destructor must not leak. Verify the object truly lives inside the holder's own bytes.

Example: `SboHolder<64> h; h.emplace<Msg>(3, 4)` constructs a `Msg` inside `h`; the object's address lies within `[&h, &h + sizeof(h))`; re-`emplace` destroys the previous `Msg` first; `reset()` and destruction run the destructor with no leak; the same holder can later hold a `long`.

hint: Since the concrete type is not known at `reset` time, stash a destroy thunk — a `void (*)(void*)` set from a captureless lambda `[](void* q){ static_cast<T*>(q)->~T(); }` (which converts to a function pointer) during `emplace`.
hint: Guard with `static_assert(sizeof(T) <= Cap)` and `static_assert(alignof(T) <= Align)` so an oversized or over-aligned type fails to compile.
hint: `emplace` must `reset()` first (destroy any current object), and the destructor is just `reset()`; make the holder non-copyable to avoid double-destroy of shared bytes.

```cpp
// starter
#include <cstddef>
#include <utility>
#include <new>
template <std::size_t Cap, std::size_t Align = alignof(std::max_align_t)>
class SboHolder {
public:
    template <typename T, typename... Args> T* emplace(Args&&... args);
    void reset();
    bool has_value() const;
    ~SboHolder();
};
```

```cpp
template <std::size_t Cap, std::size_t Align = alignof(std::max_align_t)>
class SboHolder {
    alignas(Align) unsigned char storage_[Cap];
    void (*destroy_)(void*) = nullptr;    // type-erased destructor for the current object
public:
    SboHolder() = default;
    SboHolder(const SboHolder&) = delete;             // non-copyable: bytes are unique
    SboHolder& operator=(const SboHolder&) = delete;
    ~SboHolder() { reset(); }

    template <typename T, typename... Args>
    T* emplace(Args&&... args) {
        static_assert(sizeof(T) <= Cap,   "type too large for inline storage");
        static_assert(alignof(T) <= Align, "type over-aligned for inline storage");
        reset();                                       // destroy any current object first
        T* p = ::new (static_cast<void*>(storage_)) T(std::forward<Args>(args)...);
        destroy_ = [](void* q) { static_cast<T*>(q)->~T(); };
        return p;
    }
    void reset() {
        if (destroy_) { destroy_(storage_); destroy_ = nullptr; }
    }
    bool has_value() const { return destroy_ != nullptr; }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
#include <utility>
#include <new>

struct Msg {
    static int live, dtor;
    int a, b;
    Msg(int x, int y) : a(x), b(y) { ++live; }
    ~Msg() { --live; ++dtor; }
};
int Msg::live = 0;
int Msg::dtor = 0;
//__USER__
int main() {
    {
        SboHolder<64> h;
        if (h.has_value()) { std::puts("should start empty"); return 1; }

        Msg* m = h.emplace<Msg>(3, 4);
        if (!m || !h.has_value()) { std::puts("emplace failed"); return 1; }
        if (m->a != 3 || m->b != 4) { std::puts("wrong constructed value"); return 1; }
        if (Msg::live != 1) { std::puts("live count wrong"); return 1; }

        // The object really lives inside the holder's own bytes (no heap).
        std::uintptr_t obj  = reinterpret_cast<std::uintptr_t>(m);
        std::uintptr_t self = reinterpret_cast<std::uintptr_t>(&h);
        if (!(obj >= self && obj + sizeof(Msg) <= self + sizeof(h))) { std::puts("object is not stored inline"); return 1; }

        // Re-emplace destroys the previous object first.
        h.emplace<Msg>(5, 6);
        if (Msg::dtor != 1 || Msg::live != 1) { std::puts("re-emplace should destroy the old value"); return 1; }

        // reset() runs the destructor and empties the holder.
        h.reset();
        if (h.has_value() || Msg::live != 0 || Msg::dtor != 2) { std::puts("reset should destroy and clear"); return 1; }

        // The same holder can hold a completely different type.
        SboHolder<64> h2;
        long* v = h2.emplace<long>(42);
        if (!v || *v != 42) { std::puts("should hold a long too"); return 1; }
        h2.reset();
    }
    std::puts("PASS");
}
```

**Editorial:** SBO trades a fixed byte budget for zero heap allocation: the object is placement-`new`d directly into an inline `alignas(Align) unsigned char[Cap]` buffer. The subtlety is destruction — at `reset` time the concrete type is gone, so `emplace` records a type-erased destroy thunk (`void(*)(void*)`), produced by a captureless lambda that decays to a function pointer, capturing the type without any allocation. `static_assert`s turn an oversized or over-aligned type into a compile error rather than a stack smash. The holder is non-copyable because two holders must never claim ownership of the same in-place object. This is the mechanism behind `std::function`'s small-object buffer and inline `any`/`variant`-style containers used to keep callbacks and messages off the heap in latency-critical code.
