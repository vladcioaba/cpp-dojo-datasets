## quiz: Why is the *first* write to a freshly `mmap`ed buffer slow?
tags: memory, tlb, page-fault, huge-pages
track: hft

```cpp
char* p = static_cast<char*>(mmap(/* anonymous, lazily backed */));
p[offset] = 1;   // first touch is slow
```

- [ ] The memory bus is cold and needs to warm up
- [x] The first touch triggers a page fault: the kernel maps and zeroes a physical page and fills a TLB entry; pre-fault (warm/`MAP_POPULATE`/`mlock`) and use huge pages to cut fault and TLB-miss cost
- [ ] `mmap` returns uninitialized junk that hardware must `memset` first
- [ ] The slowdown is a kernel transition on *every* access, not just the first

> Anonymous mappings are demand-paged: the virtual range exists but no physical page is backing it until you touch it. The first access faults into the kernel, which allocates and zero-fills a physical frame, updates the page tables, and populates the TLB — hundreds to thousands of cycles. Later accesses are fast until the TLB entry is evicted. Huge pages (2 MB) cover far more address space per TLB entry (fewer TLB misses), and pre-faulting during startup moves the one-time fault cost out of the hot path.
