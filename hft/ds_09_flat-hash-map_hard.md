## challenge: Open-addressing flat hash map (linear probing)
tags: allocation, hash-map, open-addressing, linear-probing, cache-locality
track: hft
difficulty: hard

A fixed-capacity integer-keyed hash map stored in flat parallel arrays, resolving collisions by linear probing rather than chaining through heap nodes. Keep a `keys`, a `vals`, and a per-slot `state` (EMPTY / OCCUPIED / DELETED). Implement `bool insert(long key, long val)` (updates in place if present, inserts otherwise, returns `false` only if the table is full and the key is absent), `long* find(long key)` (pointer to the stored value or `nullptr`), `bool erase(long key)`, and `size()`. Erase must use tombstones so it never breaks a probe chain, and insert must reuse a tombstoned slot while still detecting an existing key further along the chain.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. Keys are non-negative. No heap allocation, no `std::unordered_map`. Probing is linear: on collision, step to `(idx + 1) % N`.

Example: on `FlatHashMap<8>` with `hash(k) = k % 8`, inserting keys `1`, `9`, `17` (all hash to slot 1) lands them at slots 1, 2, 3; `find(17)` probes past the chain; `erase(9)` tombstones slot 2 but `find(17)` still succeeds; a later `insert` for a colliding key reuses the tombstoned slot.

hint: `find` walks the probe chain from `hash(key)`, stops and returns `nullptr` at the first EMPTY slot, and matches only OCCUPIED slots — it must skip DELETED slots, not stop at them.
hint: `insert` scans the chain remembering the first DELETED slot it sees; it still must continue until it either matches the key (update) or hits EMPTY (insert), only then choosing the remembered tombstone if there was one.
hint: `erase` finds the OCCUPIED slot for the key and marks it DELETED (a tombstone) rather than EMPTY, so probe chains that ran through it stay intact.

```cpp
// starter
template <size_t N>
struct FlatHashMap {
    enum State : unsigned char { EMPTY, OCCUPIED, DELETED };
    long  keys_[N];
    long  vals_[N];
    State state_[N] = {};   // all EMPTY (0)
    size_t size_ = 0;
    size_t hash(long k) const { return (size_t)k % N; }
    // implement insert / find / erase / size
};
```

```cpp
size_t size() const { return size_; }
bool insert(long k, long v) {
    size_t h = hash(k);
    size_t first_del = N;                       // sentinel: no tombstone seen
    for (size_t i = 0; i < N; ++i) {
        size_t idx = (h + i) % N;
        if (state_[idx] == EMPTY) {
            size_t put = (first_del != N) ? first_del : idx;
            keys_[put] = k; vals_[put] = v; state_[put] = OCCUPIED; ++size_;
            return true;
        }
        if (state_[idx] == DELETED) {
            if (first_del == N) first_del = idx;
        } else if (keys_[idx] == k) {           // OCCUPIED with matching key
            vals_[idx] = v;                     // update in place
            return true;
        }
    }
    if (first_del != N) {                       // table full of live+tombstones, reuse one
        keys_[first_del] = k; vals_[first_del] = v; state_[first_del] = OCCUPIED; ++size_;
        return true;
    }
    return false;                               // genuinely full, key absent
}
long* find(long k) {
    size_t h = hash(k);
    for (size_t i = 0; i < N; ++i) {
        size_t idx = (h + i) % N;
        if (state_[idx] == EMPTY) return nullptr;
        if (state_[idx] == OCCUPIED && keys_[idx] == k) return &vals_[idx];
    }
    return nullptr;
}
bool erase(long k) {
    size_t h = hash(k);
    for (size_t i = 0; i < N; ++i) {
        size_t idx = (h + i) % N;
        if (state_[idx] == EMPTY) return false;
        if (state_[idx] == OCCUPIED && keys_[idx] == k) {
            state_[idx] = DELETED; --size_; return true;
        }
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <size_t N>
struct FlatHashMap {
    enum State : unsigned char { EMPTY, OCCUPIED, DELETED };
    long  keys_[N];
    long  vals_[N];
    State state_[N] = {};
    size_t size_ = 0;
    size_t hash(long k) const { return (size_t)k % N; }
    //__USER__
};
int main() {
    FlatHashMap<8> m;
    // three keys colliding at slot 1 -> linear probe into slots 1,2,3
    if (!m.insert(1, 10) || !m.insert(9, 11) || !m.insert(17, 12)) { std::puts("collision inserts"); return 1; }
    if (m.size() != 3) { std::puts("size after inserts"); return 1; }
    long* p;
    if (!(p = m.find(1))  || *p != 10) { std::puts("find 1"); return 1; }
    if (!(p = m.find(9))  || *p != 11) { std::puts("find 9"); return 1; }
    if (!(p = m.find(17)) || *p != 12) { std::puts("find 17 through chain"); return 1; }
    if (m.find(2) != nullptr) { std::puts("find absent must be null"); return 1; }
    // update existing key, size unchanged
    if (!m.insert(1, 100) || m.size() != 3) { std::puts("update in place"); return 1; }
    if (!(p = m.find(1)) || *p != 100) { std::puts("value after update"); return 1; }
    // erase the middle of the chain -> tombstone; probing past it must still work
    if (!m.erase(9) || m.size() != 2) { std::puts("erase 9"); return 1; }
    if (m.find(9) != nullptr) { std::puts("erased key must be gone"); return 1; }
    if (!(p = m.find(17)) || *p != 12) { std::puts("find 17 past tombstone"); return 1; }
    if (m.erase(9)) { std::puts("erase of absent must return false"); return 1; }
    // insert a colliding key -> must reuse the tombstoned slot
    if (!m.insert(25, 13) || m.size() != 3) { std::puts("insert reusing tombstone"); return 1; }
    if (!(p = m.find(25)) || *p != 13) { std::puts("find reused key"); return 1; }
    if (!(p = m.find(17)) || *p != 12) { std::puts("chain intact after reuse"); return 1; }
    // fill to capacity, then a fresh key must be rejected
    if (!m.insert(2, 20) || !m.insert(3, 30) || !m.insert(4, 40) ||
        !m.insert(5, 50) || !m.insert(6, 60)) { std::puts("fill to capacity"); return 1; }
    if (m.size() != 8) { std::puts("should be full at 8"); return 1; }
    if (m.insert(99, 990)) { std::puts("insert into full table must return false"); return 1; }
    if (m.find(99) != nullptr) { std::puts("rejected key must be absent"); return 1; }
    // updating an existing key in a full table must still work
    if (!m.insert(4, 400) || m.size() != 8) { std::puts("update in full table"); return 1; }
    if (!(p = m.find(4)) || *p != 400) { std::puts("value after full-table update"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A flat, open-addressed hash map keeps everything in contiguous arrays and resolves collisions by walking to the next slot, so a lookup streams through cache lines instead of dereferencing a chain of heap nodes the way `std::unordered_map` does (one allocation per element, pointer-chasing per probe, poor locality). The subtlety is deletion: you cannot blank a slot to EMPTY, because that would terminate a probe chain that legitimately runs through it and hide keys inserted later — so you plant a DELETED tombstone that `find` skips and `insert` may reclaim. `insert` therefore can't stop at the first tombstone; it remembers it but keeps probing so it still detects an existing key further down the chain, and only falls back to the tombstone when it reaches an EMPTY terminator. Linear probing (step +1) maximizes cache-line reuse; the price is tombstone buildup, which production maps bound with a load-factor cap and periodic rehash. Average operations are O(1); zero heap traffic on the hot path.
