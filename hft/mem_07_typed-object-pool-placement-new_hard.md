## challenge: Typed object pool with placement new
tags: object-pool, placement-new, construct-destroy
track: hft
difficulty: hard

Raw byte allocators hand back untyped storage; a *typed* pool also runs constructors and destructors correctly. Implement `ObjectPool<T, N>`: `N` slots of raw, `T`-aligned storage plus an intrusive free list. `create(args...)` pops a slot and constructs a `T` in place with placement `new` (or returns `nullptr` when full); `destroy(p)` runs `p->~T()` exactly once and recycles the slot. Never construct into occupied storage and never leak a destructor call.

Constraints: no heap allocation — storage is a fixed inline array of `union` slots; construction is perfectly forwarded; `create` returns `nullptr` only when all `N` slots are live. Freeing then re-creating must reuse the slot.

Example: an `ObjectPool<Widget, 3>` constructs three `Widget`s (three ctor calls, three live); the fourth `create` returns `nullptr`; `destroy` on one runs exactly one destructor; a following `create` reuses that slot; when all are destroyed the live count is zero and destructors ran once each.

hint: Make each slot a `union` of a `Slot* next` (free-list link) and `alignas(T) unsigned char storage[sizeof(T)]` — a free slot holds the link, a live slot holds the object, sharing the same bytes.
hint: `create` = pop the free list, then `::new (slot->storage) T(std::forward<Args>(args)...)`; `destroy` = call `p->~T()`, then push the slot (recovered via `reinterpret_cast<Slot*>(p)`) back on the free list.
hint: A `union` member's address equals the slot's address, so `reinterpret_cast<Slot*>(p)` recovers the owning slot from the object pointer.

```cpp
// starter
#include <cstddef>
#include <utility>
#include <new>
template <typename T, std::size_t N>
class ObjectPool {
public:
    template <typename... Args> T* create(Args&&... args); // nullptr if full
    void destroy(T* p);                                    // run ~T, recycle slot
};
```

```cpp
template <typename T, std::size_t N>
class ObjectPool {
    union Slot {
        Slot* next;                              // link when free
        alignas(T) unsigned char storage[sizeof(T)];  // the object when live
    };
    Slot slots_[N];
    Slot* free_ = nullptr;
public:
    ObjectPool() {
        for (std::size_t i = 0; i < N; ++i) {    // thread all slots onto the free list
            slots_[i].next = free_;
            free_ = &slots_[i];
        }
    }
    template <typename... Args>
    T* create(Args&&... args) {
        if (!free_) return nullptr;
        Slot* s = free_;
        free_ = free_->next;
        return ::new (static_cast<void*>(s->storage)) T(std::forward<Args>(args)...);
    }
    void destroy(T* p) {
        if (!p) return;
        p->~T();                                 // run the destructor exactly once
        Slot* s = reinterpret_cast<Slot*>(p);    // object address == slot address
        s->next = free_;
        free_ = s;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <utility>
#include <new>

struct Widget {
    static int live, ctor, dtor;
    int id;
    explicit Widget(int i) : id(i) { ++live; ++ctor; }
    ~Widget() { --live; ++dtor; }
};
int Widget::live = 0;
int Widget::ctor = 0;
int Widget::dtor = 0;
//__USER__
int main() {
    {
        ObjectPool<Widget, 3> pool;
        Widget* a = pool.create(1);
        Widget* b = pool.create(2);
        Widget* c = pool.create(3);
        if (!a || !b || !c) { std::puts("create returned null too early"); return 1; }
        if (Widget::live != 3 || Widget::ctor != 3) { std::puts("wrong ctor/live count"); return 1; }
        if (a->id != 1 || b->id != 2 || c->id != 3) { std::puts("wrong constructed values"); return 1; }

        // Pool exhausted.
        if (pool.create(4) != nullptr) { std::puts("should be full"); return 1; }

        // destroy runs the destructor exactly once.
        pool.destroy(b);
        if (Widget::live != 2 || Widget::dtor != 1) { std::puts("wrong dtor/live count"); return 1; }

        // The recycled slot is handed out again.
        Widget* d = pool.create(5);
        if (!d) { std::puts("should reuse a freed slot"); return 1; }
        if (d != b) { std::puts("should reuse b's exact slot"); return 1; }
        if (Widget::live != 3 || d->id != 5) { std::puts("reuse state wrong"); return 1; }

        pool.destroy(a);
        pool.destroy(c);
        pool.destroy(d);
        if (Widget::live != 0 || Widget::dtor != 4) { std::puts("final counts wrong"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The `union` slot is the crux: a free slot uses its bytes as a `Slot*` free-list link, a live slot uses the same bytes as `T` storage — no per-slot flag, no wasted space. `create` pops a slot and uses placement `new` to run `T`'s constructor into that raw, correctly aligned storage (`alignas(T)` guarantees alignment; perfect forwarding preserves the arguments). `destroy` manually invokes `~T()` — because placement `new` was used, you must destruct explicitly and exactly once — then recovers the owning slot (a union member shares the slot's address) and pushes it back. The result is O(1) typed allocation with proper object lifetime and zero heap traffic: the hand-rolled cousin of a `std::pmr` monotonic/pool resource.
