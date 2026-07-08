## challenge: Fixed-size free-list pool allocator
tags: freelist, pool, intrusive-list
track: hft
difficulty: medium

When every object is the same size, a free-list pool gives O(1) alloc and free with zero fragmentation. Implement `Pool` over a caller-supplied buffer split into `count` blocks of `block_size` bytes. The trick: thread a singly linked "free list" through the free blocks themselves — store each block's `next` pointer in the block's own memory (intrusive), so the pool needs no side table. `alloc()` pops the head block (or `nullptr` when empty); `free(p)` pushes a block back onto the head.

Constraints: `block_size >= sizeof(void*)` and the buffer is suitably aligned; `alloc`/`free` are O(1); `alloc()` returns `nullptr` only when every block is handed out. A freed block, re-allocated, must come back (LIFO).

Example: with `block_size = 32, count = 4`, four `alloc()` calls hand out four distinct in-range blocks, the fifth returns `nullptr`; after `free(b)` the next `alloc()` returns exactly `b`.

hint: Reinterpret each free block as a node holding a single `next` pointer — the free-list links live inside the free memory itself, costing no extra storage.
hint: In the constructor, walk the blocks and push every one onto the list; `alloc` = pop head, `free` = push onto head. Both are a few pointer assignments.
hint: `alloc` must return `nullptr` when the head is null; don't dereference an empty list.

```cpp
// starter
#include <cstddef>
class Pool {
public:
    Pool(void* buf, std::size_t block_size, std::size_t count);
    void* alloc();          // nullptr when exhausted
    void  free(void* p);    // return a block to the pool
};
```

```cpp
class Pool {
    struct Node { Node* next; };
    Node* head_ = nullptr;
public:
    Pool(void* buf, std::size_t block_size, std::size_t count) {
        char* p = static_cast<char*>(buf);
        for (std::size_t i = 0; i < count; ++i) {
            Node* node = reinterpret_cast<Node*>(p + i * block_size);
            node->next = head_;   // intrusive link stored in the block itself
            head_ = node;
        }
    }
    void* alloc() {
        if (!head_) return nullptr;
        Node* n = head_;
        head_ = head_->next;
        return n;
    }
    void free(void* p) {
        Node* n = static_cast<Node*>(p);
        n->next = head_;
        head_ = n;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
#include <cstdint>
//__USER__
int main() {
    constexpr std::size_t BS = 32, N = 4;
    alignas(16) unsigned char buffer[BS * N];
    Pool pool(buffer, BS, N);

    void* blocks[N];
    std::uintptr_t base = reinterpret_cast<std::uintptr_t>(buffer);
    for (std::size_t i = 0; i < N; ++i) {
        blocks[i] = pool.alloc();
        if (!blocks[i]) { std::puts("alloc returned null too early"); return 1; }
        std::uintptr_t addr = reinterpret_cast<std::uintptr_t>(blocks[i]);
        if (addr < base || addr + BS > base + BS * N) { std::puts("block out of range"); return 1; }
    }
    // All blocks handed out.
    if (pool.alloc() != nullptr) { std::puts("should be exhausted"); return 1; }

    // Blocks are distinct.
    for (std::size_t i = 0; i < N; ++i)
        for (std::size_t j = i + 1; j < N; ++j)
            if (blocks[i] == blocks[j]) { std::puts("duplicate block handed out"); return 1; }

    // free then alloc returns the same block (LIFO).
    pool.free(blocks[2]);
    void* again = pool.alloc();
    if (again != blocks[2]) { std::puts("free/alloc should recycle the block"); return 1; }
    if (pool.alloc() != nullptr) { std::puts("should be exhausted again"); return 1; }

    std::puts("PASS");
}
```

**Editorial:** Because every block is identical in size, the pool stores its bookkeeping *inside the free blocks*: each free block begins with a `next` pointer, forming an intrusive singly linked list. Allocation pops the head; deallocation pushes onto the head — both O(1) with no scanning and no external metadata, so there is zero fragmentation and excellent locality. The only requirements are that `block_size` is at least `sizeof(void*)` and the buffer is aligned for a pointer. This is the canonical order-router / message pool: fixed object size, allocate and free millions of times per second on the hot path.
