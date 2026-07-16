## challenge: memo cache that doesn't pin its entries
tags: smart-pointers, raii
track: core
difficulty: medium

`TextureCache::get(name)` must return a shared texture: if a previously returned texture for that name is still alive *anywhere*, return that same object without loading again; if every user has released it, load it fresh and re-cache. The cache itself must never keep a texture alive — that's why the map stores `std::weak_ptr`. The harness counts loads and checks object identity through both phases.

hint: `cache_[name]` gives you the weak slot (creating an empty one on first use). A `weak_ptr` can't be dereferenced — convert it to an owning pointer first.
hint: `slot.lock()` returns a `shared_ptr`: non-null means the texture is still alive somewhere — return it, no load. Null means it expired (or was never loaded).
hint: On a miss, `std::make_shared<Texture>(name)`, assign it into the weak slot (shared-to-weak assignment just works), and return the shared_ptr — the CALLER becomes the owner, the cache only observes.

```cpp
// starter
struct Texture {
    inline static int loads = 0;    // counts real (re)loads
    std::string name;
    explicit Texture(std::string n) : name(std::move(n)) { ++loads; }
};

class TextureCache {
public:
    // TODO: if a texture for `name` is still alive, return that same object
    // (no new load). Otherwise load it, remember it weakly, return it.
    std::shared_ptr<Texture> get(const std::string& name) {
        return std::make_shared<Texture>(name);   // always reloads — wrong
    }

private:
    // weak_ptr on purpose: the cache must not keep textures alive.
    std::unordered_map<std::string, std::weak_ptr<Texture>> cache_;
};
```

```cpp
struct Texture {
    inline static int loads = 0;    // counts real (re)loads
    std::string name;
    explicit Texture(std::string n) : name(std::move(n)) { ++loads; }
};

class TextureCache {
public:
    std::shared_ptr<Texture> get(const std::string& name) {
        std::weak_ptr<Texture>& slot = cache_[name];
        if (std::shared_ptr<Texture> existing = slot.lock()) {
            return existing;                    // still alive: same object
        }
        auto fresh = std::make_shared<Texture>(name);
        slot = fresh;                           // observe, don't own
        return fresh;
    }

private:
    // weak_ptr on purpose: the cache must not keep textures alive.
    std::unordered_map<std::string, std::weak_ptr<Texture>> cache_;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    TextureCache cache;

    auto a = cache.get("grass");
    assert(a != nullptr);
    assert(a->name == "grass");
    assert(Texture::loads == 1);

    auto b = cache.get("grass");        // alive -> the very same object
    assert(b == a);
    assert(Texture::loads == 1);

    auto c = cache.get("rock");         // different key -> real load
    assert(c != a);
    assert(Texture::loads == 2);

    a.reset();
    b.reset();                          // last "grass" owner gone -> expires
    auto d = cache.get("grass");        // must reload
    assert(Texture::loads == 3);
    assert(d->name == "grass");

    auto e = cache.get("rock");         // c still owns it -> cached hit
    assert(e == c);
    assert(Texture::loads == 3);

    std::puts("PASS");
}
```

**Editorial:** The design question in any cache is who owns the entries. Store `shared_ptr` and the cache is an owner: every texture ever requested stays resident until the cache is torn down — for assets, that's an unbounded memory pin dressed up as an optimization. Storing `weak_ptr` inverts it: the *users* own textures; the cache merely remembers where the live ones are. `lock()` is the pivot — it atomically checks liveness and, in the same operation, produces an owning `shared_ptr`, so the object can't die between the check and the use (the `expired()`-then-`lock()` two-step is a TOCTOU bug in concurrent code; lock once and test the result). The lifecycle in the harness is the whole contract: while `a`/`b` hold "grass", repeat lookups return the identical object with no load; when both release, the weak slot expires and the next `get` pays for a reload — deduplication while hot, natural eviction when cold. One production refinement worth knowing: expired `weak_ptr`s linger as map entries until overwritten, so long-running caches occasionally sweep (`erase` entries where `slot.expired()`). Same pattern, same reasoning, powers interned strings, connection registries, and observer lists that must not extend subscriber lifetimes.
