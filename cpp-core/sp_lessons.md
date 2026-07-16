# Smart pointers

8 tutorial cards, in reading order: ownership and RAII first, then `unique_ptr`, `shared_ptr` and its satellites, and finally how to design APIs around them.

## fact: Ownership is the question; RAII is the answer
tags: smart-pointers, raii, ownership
track: core

Every heap object needs exactly one answer to the question "who deletes this, and when?" Raw `new`/`delete` leaves that answer in the programmer's head: every early `return`, thrown exception, and forgotten error path is a chance to leak, and every second `delete` on the same pointer is undefined behavior. The failure mode isn't carelessness — it is that manual cleanup must be re-proven correct on *every* exit path, forever, through every refactor.

RAII (Resource Acquisition Is Initialization) moves the answer into the type system: acquire the resource in a constructor, release it in the destructor, and let the compiler insert the cleanup on every path — normal returns, exceptions, `break`, everything. Smart pointers are simply RAII applied to heap memory, with the ownership *policy* encoded in the type: `std::unique_ptr` says "exactly one owner," `std::shared_ptr` says "last owner out turns off the lights," `std::weak_ptr` says "I only observe."

That is why raw `new`/`delete` died in application code: not because they are slow, but because they encode ownership nowhere. Modern guideline: raw pointers still exist, but only as non-owning observers — anything that *owns* goes in a smart pointer.

```cpp
void risky() {
    auto buf = std::make_unique<std::vector<int>>(1024);
    parse(*buf);          // may throw — buf still freed
    if (!valid(*buf)) return;  // early exit — buf still freed
}                          // normal exit — buf freed. Zero delete statements.
```

## fact: unique_ptr — ownership as a move-only type
tags: smart-pointers, raii, unique-ptr
track: core

`std::unique_ptr<T>` is sole ownership made compilable. Copying is deleted — two `unique_ptr`s owning one object is a double delete waiting to happen, so the compiler simply rejects it. Transfer is explicit: `std::move` hands the pointer over and leaves the source null. With the default deleter it is exactly the size of a raw pointer and `operator*`/`operator->` compile to the same code, so there is no excuse not to use it.

The escape hatches are precise:

- `get()` — borrow the raw pointer, ownership unchanged. For passing to observers.
- `release()` — give up ownership and return the raw pointer; the caller is now manually responsible. For handing off to C APIs.
- `reset(p)` — destroy the current object (if any), then own `p`. `reset()` alone just destroys.

Mixing up `get()` and `release()` is a classic bug: `get()` into something that deletes double-frees; `release()` into something that doesn't delete leaks.

There is also an array form, `std::unique_ptr<T[]>`, which calls `delete[]` and offers `operator[]` — though `std::vector` or `std::array` is almost always the better choice.

```cpp
std::unique_ptr<Widget> a = std::make_unique<Widget>();
// auto b = a;              // error: copy is deleted
auto b = std::move(a);      // a is now nullptr
Widget* view = b.get();     // borrow, b still owns
b.reset();                  // destroyed here, deterministically
```

## fact: make_unique vs new — and the make_shared allocation myth
tags: smart-pointers, raii, make-unique
track: core

Prefer `std::make_unique<T>(args...)` over `std::unique_ptr<T>(new T(args...))` for three reasons. First, exception safety: before C++17, an expression like `f(std::unique_ptr<T>(new T), g())` could evaluate `new T`, then run `g()`, and if `g()` threw, the not-yet-wrapped object leaked. C++17 banned that interleaving, but `make_unique` was the fix that never needed a language change. Second, greppability: a codebase using `make_unique` everywhere has *zero* naked `new` expressions, so any `new` that appears in review is automatically suspicious. Third, it says the type once instead of twice.

Now the myth to unlearn: `make_unique` performs **one** allocation — exactly like `new`. There is no fusion trick, because `unique_ptr` has no control block. The famous "single allocation" optimization belongs to `std::make_shared`, which allocates the object and the shared control block in one contiguous chunk instead of two separate allocations. That is a real win — better locality, half the allocator traffic — with one subtle cost: the object's *storage* cannot be freed until the last `weak_ptr` dies too, since object and control block share one block. For huge objects outlived by long-lived `weak_ptr`s, separate allocation via `shared_ptr<T>(new T)` can actually be the right call.

```cpp
auto u = std::make_unique<Config>("a.toml");   // 1 allocation (same as new)
auto s = std::make_shared<Config>("a.toml");   // 1 fused allocation
std::shared_ptr<Config> t(new Config("a.toml")); // 2: object + control block
```

## fact: shared_ptr internals — the control block
tags: smart-pointers, raii, shared-ptr
track: core

A `shared_ptr<T>` is two raw pointers wide: one to the object, one to a heap-allocated **control block**. The control block holds the *strong* count (owners), the *weak* count (observers), and, type-erased, the deleter and allocator. Copying a `shared_ptr` atomically increments the strong count; destroying one decrements it. Strong count hitting zero destroys the object; the control block itself lives on until the weak count also drains, because `weak_ptr`s need somewhere to look to answer "expired?".

Two consequences follow directly. Those increments are *atomic* read-modify-writes — thread-safe, but each copy is a contended RMW plus potential cache-line ping-pong, which is why hot loops pass `const shared_ptr&` or a raw pointer instead of copying. And the count lives in the control block, not the pointer — so two `shared_ptr`s constructed independently from the same raw pointer create two control blocks, each convinced it is the sole owner: double delete.

The oddest constructor is the **aliasing constructor**: `shared_ptr<M>(owner, &owner->member)` *points* at the member but shares the owner's control block, keeping the whole parent alive. `use_count()` reports the strong count; treat it as a debugging aid, not synchronization.

```cpp
auto owner = std::make_shared<Widget>();
std::shared_ptr<int> id(owner, &owner->id); // aliasing: points at the int,
owner.reset();                              // ...but owns the Widget
assert(id.use_count() == 1);                // Widget still alive via id
```

## fact: weak_ptr — observing without owning
tags: smart-pointers, raii, weak-ptr
track: core

`std::weak_ptr<T>` references a `shared_ptr`-managed object without keeping it alive. It bumps only the control block's weak count, so the object dies on schedule; the `weak_ptr` can then tell you it's gone instead of dangling. That single property solves two problems `shared_ptr` creates for itself.

**Cycles.** If a parent holds `shared_ptr<Child>` and the child holds `shared_ptr<Parent>` back, each keeps the other's strong count above zero and neither destructor ever runs — a leak no tool but a heap profiler will show you. Rule: owning edges point one way (parent→child `shared_ptr`); back-edges observe (child→parent `weak_ptr`).

**Caches and observers.** A cache holding `shared_ptr` pins every entry alive forever. A cache of `weak_ptr` lets entries die naturally when the last real user releases them, and `lock()` tells the cache whether a revival is needed.

The access pattern is always the same and is race-free by construction: `lock()` atomically produces an owning `shared_ptr` — non-null only if the object is still alive — and you use *that*, never the `weak_ptr` directly. Checking `expired()` and then locking is a TOCTOU bug in concurrent code; just `lock()` and test.

```cpp
if (auto sp = cache_entry.lock()) {
    use(*sp);          // alive — sp keeps it so until end of scope
} else {
    sp = reload();     // died — rebuild and re-cache
}
```

## fact: enable_shared_from_this — why shared_ptr(this) double-frees
tags: smart-pointers, raii, enable-shared-from-this
track: core

Inside a member function, `this` is a raw pointer — and sometimes the object needs to hand out shared ownership of itself: an async callback, a task queue, a subscriber list. The reflex `std::shared_ptr<Widget>(this)` is a landmine: `shared_ptr`'s raw-pointer constructor *always creates a brand-new control block*. The object is already owned by some other `shared_ptr` with its own control block; now two independent bookkeepers each hold strong count 1, and each will call `delete` when it drains. Double free.

`std::enable_shared_from_this<T>` is the fix. Inherit from it (publicly — the machinery needs to see the base), and the first `shared_ptr` created for the object secretly stores a `weak_ptr` to itself inside the base. `shared_from_this()` then locks that hidden `weak_ptr`, returning a `shared_ptr` that shares the *original* control block — one owner group, correct counts.

Two rules: the object must already be managed by a `shared_ptr` when you call `shared_from_this()` — calling it on a stack object or inside the constructor (before any `shared_ptr` exists) is undefined behavior pre-C++17 and throws `std::bad_weak_ptr` since. And factories help enforce this: make the constructor private and expose `static std::shared_ptr<Widget> create()`.

```cpp
struct Task : std::enable_shared_from_this<Task> {
    void schedule(Queue& q) {
        q.push([self = shared_from_this()] { self->run(); }); // co-owner
    }   // NOT std::shared_ptr<Task>(this) — that double-frees
};
```

## fact: Custom deleters — and the pImpl connection
tags: smart-pointers, raii, deleters
track: core

Smart pointers manage more than memory: anything with an acquire/release pair — `FILE*`, sockets, C library handles — fits, via a custom deleter. The two pointer types treat deleters very differently.

For `unique_ptr` the deleter is part of the **type**: `std::unique_ptr<FILE, decltype(&fclose)>` and `std::unique_ptr<FILE>` are unrelated types. A function-pointer deleter also doubles the size (the pointer must be stored). Prefer a stateless functor — `struct FCloser { void operator()(FILE* f) const { fclose(f); } };` — which empty-base-optimizes away, keeping the handle pointer-sized and the deleter un-forgettable.

For `shared_ptr` the deleter is **type-erased into the control block**: `std::shared_ptr<FILE>(fopen(...), &fclose)` is just `shared_ptr<FILE>`. Different deleters, same type — flexible, at the cost of the control block indirection.

The deleter-is-part-of-the-type fact powers the pImpl idiom: `std::unique_ptr<Impl> impl_` with `Impl` forward-declared compiles fine, *until* something instantiates the deleter — which requires `Impl` to be complete. The compiler-generated destructor in the header does exactly that. The fix is one line of ritual: declare `~Widget();` in the header, define `Widget::~Widget() = default;` in the .cpp after `Impl`'s full definition. Same for the move operations.

```cpp
struct FCloser { void operator()(FILE* f) const { std::fclose(f); } };
using FilePtr = std::unique_ptr<FILE, FCloser>;   // still pointer-sized
FilePtr open_log() { return FilePtr(std::fopen("log.txt", "r")); }
```

## fact: API design — sinks, observers, and when NOT to use shared_ptr
tags: smart-pointers, raii, api-design
track: core

Smart pointers in signatures are ownership statements — so only put one there when the function participates in ownership.

**Sink**: take `std::unique_ptr<T>` *by value*. The caller must `std::move` in, the transfer is visible at the call site, and the signature is honest: "I consume this." **Factory**: return `std::unique_ptr<T>` by value — cheap (a pointer move), and it converts implicitly to `shared_ptr<T>` if the caller wants sharing, so returning `unique_ptr` never forecloses anything. Returning `shared_ptr` does: you cannot go back. **Observer**: a function that just *uses* the object takes `T&` (never null) or `T*` (maybe null) — never a smart pointer. Passing `shared_ptr` by value here costs two atomic RMWs and, worse, lies about the contract. `const shared_ptr<T>&` is only for functions that might *copy* the pointer conditionally.

`shared_ptr` is the correct tool only when lifetime is genuinely unknowable at design time — multiple independent consumers, any of which may be last to finish. That is rarer than it looks. Reaching for it "to be safe" buys atomic traffic, control-block allocations, cycle risk, and — the real cost — code where *nobody* can say who owns what. Default to `unique_ptr`; escalate to `shared_ptr` only under evidence.

```cpp
std::unique_ptr<Conn> make_conn(Config);      // factory
void adopt(std::unique_ptr<Conn> c);          // sink: caller moves in
void log_stats(const Conn& c);                // observer: no ownership
```
