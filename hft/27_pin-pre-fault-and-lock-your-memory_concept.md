## fact: Pin, pre-fault, and lock your memory
tags: numa, page-faults, mlock
track: hft

On multi-socket boxes memory is **NUMA**: reaching another socket's RAM costs extra. Linux allocates a page on the node of the thread that **first touches** it, so allocate *and* initialize on the core that will use it, and pin threads (`taskset`/`sched_setaffinity`, plus isolated cores) so they don't migrate off-node.

Two more startup rituals: the first touch of fresh memory takes a **page fault** into the kernel (a *minor* fault, or *major* if it hits disk) — so **pre-fault** by writing every page up front. And `mlock`/`mlockall` pins pages in RAM so nothing is swapped out mid-trade. The pattern: allocate, pre-fault, `mlock`, pin threads, then **warm caches and branch predictors** with dummy iterations before the open — pay every one-time cost before it can hurt.
