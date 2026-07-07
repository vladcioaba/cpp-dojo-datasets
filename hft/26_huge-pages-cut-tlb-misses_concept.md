## fact: Huge pages cut TLB misses
tags: tlb, huge-pages, memory
track: hft

Virtual→physical translation goes through the **TLB**, a small cache of page mappings. A TLB miss triggers a multi-level **page walk** (extra memory accesses). With 4 KB pages, a large working set blows the TLB and every stride pays for a walk.

**Huge pages** (2 MB or 1 GB on x86-64) map far more memory per TLB entry, so a big buffer is covered by a handful of entries — fewer misses, fewer walks. Use `MADV_HUGEPAGE`, explicit hugetlbfs, or reserve at boot, and pre-fault so the pages are resident before the hot path runs. The win shows up for large, randomly-accessed structures (order books, big hash tables), not tight sequential scans the prefetcher already handles.
