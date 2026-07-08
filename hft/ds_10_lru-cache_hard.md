## challenge: Fixed-capacity O(1) LRU cache
tags: allocation, lru-cache, intrusive-list, hash-map, cache-locality
track: hft
difficulty: hard

A least-recently-used cache with a compile-time capacity and no dynamic allocation, offering O(1) `get` and `put`. Combine two flat structures over the same node slots: a doubly-linked recency list (most-recently-used at the head, least-recently-used at the tail) threaded through `prev_`/`next_` index arrays, and an open-addressed key index for O(1) lookup. Implement `bool get(long key, long& out)` (on hit, copy the value and promote the key to most-recently-used) and `void put(long key, long val)` (insert or update, promoting to MRU; when full, evict the tail before inserting). On a miss `get` returns `false`.

Constraints: capacity `N` is a compile-time constant, `1 <= N`. Keys are non-negative. No heap allocation, no `std::` containers. `get`/`put` are O(1): list splices are pointer (index) updates and lookups go through the key index. Evictions remove exactly the least-recently-used entry.

Example: on `LRUCache<3>`, `put(1,10); put(2,20); put(3,30)` fills it (MRU order 3,2,1); `get(1)` returns `10` and promotes `1`; `put(4,40)` now evicts `2` (the LRU); a later `get(2)` returns `false`.

hint: Keep the recency order as an intrusive doubly-linked list over index arrays — promoting a node is `detach` then `attach_front`; evicting is `detach(tail_)`.
hint: When not full, take the next unused slot (`size_`); when full, reuse the evicted tail's slot for the new key so you never allocate.
hint: The key-to-slot index is a linear-probing hash sized larger than `N` (so it always has an empty slot); on eviction remove the old key from it with backward-shift deletion so no tombstones accumulate.

```cpp
// starter
template <size_t N>
struct LRUCache {
    static constexpr int NIL = -1;
    static constexpr size_t M = 2 * N;          // key index slots (load factor < 1)
    static constexpr size_t MNIL = ~size_t(0);
    long key_[N];
    long val_[N];
    int  prev_[N];
    int  next_[N];
    int  head_ = NIL;      // MRU
    int  tail_ = NIL;      // LRU
    size_t size_ = 0;
    long   mkey_[M];
    int    mslot_[M];
    bool   mocc_[M] = {};
    size_t mhash(long k) const { return (size_t)k % M; }
    // implement get / put (plus any private list/index helpers you need)
};
```

```cpp
// ---- intrusive recency list over index arrays ----
void detach(int s) {
    int p = prev_[s], n = next_[s];
    if (p != NIL) next_[p] = n; else head_ = n;
    if (n != NIL) prev_[n] = p; else tail_ = p;
}
void attach_front(int s) {
    prev_[s] = NIL;
    next_[s] = head_;
    if (head_ != NIL) prev_[head_] = s; else tail_ = s;
    head_ = s;
}
void move_to_front(int s) {
    if (head_ == s) return;
    detach(s);
    attach_front(s);
}
// ---- linear-probing key index (key -> node slot) ----
size_t map_find(long k) const {
    size_t idx = mhash(k);
    for (size_t i = 0; i < M; ++i) {
        if (!mocc_[idx]) return MNIL;
        if (mkey_[idx] == k) return idx;
        idx = (idx + 1) % M;
    }
    return MNIL;
}
void map_insert(long k, int slot) {
    size_t idx = mhash(k);
    while (mocc_[idx]) idx = (idx + 1) % M;     // guaranteed to terminate: size_ < M
    mocc_[idx] = true; mkey_[idx] = k; mslot_[idx] = slot;
}
void map_erase_at(size_t i) {                   // backward-shift deletion, no tombstones
    while (true) {
        mocc_[i] = false;
        size_t j = i;
        while (true) {
            j = (j + 1) % M;
            if (!mocc_[j]) return;
            size_t k = mhash(mkey_[j]);
            bool keep = (i <= j) ? (i < k && k <= j) : (i < k || k <= j);
            if (!keep) break;                   // entry j can slide back to fill i
        }
        mkey_[i] = mkey_[j]; mslot_[i] = mslot_[j]; mocc_[i] = true;
        i = j;
    }
}
// ---- public API ----
bool get(long k, long& out) {
    size_t p = map_find(k);
    if (p == MNIL) return false;
    int s = mslot_[p];
    out = val_[s];
    move_to_front(s);
    return true;
}
void put(long k, long v) {
    size_t p = map_find(k);
    if (p != MNIL) {                            // update existing
        int s = mslot_[p];
        val_[s] = v;
        move_to_front(s);
        return;
    }
    int s;
    if (size_ < N) {
        s = (int)size_++;                       // next unused slot
    } else {
        s = tail_;                              // evict LRU, reuse its slot
        map_erase_at(map_find(key_[s]));
        detach(s);
    }
    key_[s] = k; val_[s] = v;
    map_insert(k, s);
    attach_front(s);
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
using std::size_t;
template <size_t N>
struct LRUCache {
    static constexpr int NIL = -1;
    static constexpr size_t M = 2 * N;
    static constexpr size_t MNIL = ~size_t(0);
    long key_[N];
    long val_[N];
    int  prev_[N];
    int  next_[N];
    int  head_ = NIL;
    int  tail_ = NIL;
    size_t size_ = 0;
    long   mkey_[M];
    int    mslot_[M];
    bool   mocc_[M] = {};
    size_t mhash(long k) const { return (size_t)k % M; }
    //__USER__
};
int main() {
    long out;
    // ---- scenario A: recency + eviction ----
    LRUCache<3> c;
    if (c.get(1, out)) { std::puts("get on empty must miss"); return 1; }
    c.put(1, 10); c.put(2, 20); c.put(3, 30);   // MRU order: 3,2,1
    if (!c.get(1, out) || out != 10) { std::puts("get 1"); return 1; }     // promotes 1 -> MRU: 1,3,2
    c.put(4, 40);                               // evicts LRU (2); MRU: 4,1,3
    if (c.get(2, out)) { std::puts("2 must be evicted"); return 1; }
    if (!c.get(1, out) || out != 10) { std::puts("1 must survive"); return 1; }
    if (!c.get(3, out) || out != 30) { std::puts("3 must survive"); return 1; }   // promote 3
    if (!c.get(4, out) || out != 40) { std::puts("4 must survive"); return 1; }   // MRU: 4,3,1
    c.put(5, 50);                               // LRU is 1 -> evict 1
    if (c.get(1, out)) { std::puts("1 must be evicted"); return 1; }
    if (!c.get(3, out) || out != 30) { std::puts("3 still here"); return 1; }
    if (!c.get(4, out) || out != 40) { std::puts("4 still here"); return 1; }
    if (!c.get(5, out) || out != 50) { std::puts("5 still here"); return 1; }
    // update existing value must not evict, must promote
    c.put(3, 333);
    if (!c.get(3, out) || out != 333) { std::puts("update value"); return 1; }

    // ---- scenario B: key index collisions + backward-shift on evict ----
    LRUCache<4> d;                              // M = 8
    d.put(2, 20); d.put(10, 100); d.put(18, 180); d.put(26, 260);  // all hash to slot 2 mod 8
    if (!d.get(2, out)  || out != 20)  { std::puts("collide find 2"); return 1; }
    if (!d.get(10, out) || out != 100) { std::puts("collide find 10"); return 1; }
    if (!d.get(18, out) || out != 180) { std::puts("collide find 18"); return 1; }
    if (!d.get(26, out) || out != 260) { std::puts("collide find 26"); return 1; }
    d.put(34, 340);                             // full -> evict LRU (2), backward-shift its key out
    if (d.get(2, out)) { std::puts("2 must be evicted from collision chain"); return 1; }
    if (!d.get(10, out) || out != 100) { std::puts("10 findable after shift"); return 1; }
    if (!d.get(18, out) || out != 180) { std::puts("18 findable after shift"); return 1; }
    if (!d.get(26, out) || out != 260) { std::puts("26 findable after shift"); return 1; }
    if (!d.get(34, out) || out != 340) { std::puts("34 findable after shift"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** An O(1) LRU needs two structures that agree on the same entries: a recency order and a key lookup. Both are built flat over fixed index arrays with no per-node allocation. The recency order is an intrusive doubly-linked list threaded through `prev_`/`next_` (indices, not pointers), so promoting an entry to most-recently-used or evicting the least-recently-used tail is a handful of index writes — versus `std::list`, which would heap-allocate a node per entry. The lookup is an open-addressed linear-probing index from key to node slot; using backward-shift deletion on eviction keeps it tombstone-free, so the probe chains for colliding keys stay short and correct even as entries churn. Crucially, an eviction and the following insert reuse the same physical slot, so a full cache does steady-state work with zero heap traffic and a compact, cache-resident footprint — the difference between a predictable few-nanosecond `get` and an allocator-bound one on the hot path.
