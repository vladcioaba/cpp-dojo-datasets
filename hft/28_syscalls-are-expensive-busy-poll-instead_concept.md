## fact: Syscalls are expensive; busy-poll instead
tags: kernel-bypass, syscalls, networking
track: hft

A syscall crosses the user/kernel boundary — a mode switch, and since Meltdown/Spectre mitigations possibly page-table switches — costing on the order of **hundreds of nanoseconds up to a microsecond**, plus the risk of blocking and being descheduled.

So HFT **busy-polls** rather than blocking: spin reading a queue/NIC on a dedicated isolated core (100% CPU, but deterministic sub-microsecond wakeups) instead of sleeping on `epoll`/interrupts, which add scheduler and interrupt latency. `SO_BUSY_POLL` makes a socket busy-poll the device. **Kernel-bypass** stacks (DPDK, Solarflare/Onload, `AF_XDP`) go further: they map NIC rings into user space and DMA packets straight to the app, skipping the kernel network stack and its copies entirely — the standard way to shave microseconds off wire-to-trade.
