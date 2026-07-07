## fact: Kernel bypass vs full hardware
tags: hft, latency, architecture
track: fpga

**Kernel bypass** (Solarflare Onload, DPDK) removes the OS network stack from the path, delivering packets straight to userspace. It cuts latency dramatically versus ordinary sockets, but the logic still runs as software on a CPU — so jitter and single-digit-microsecond latency remain.

**Full hardware** goes further: parsing, decision, and order generation happen entirely in the FPGA, and a matching packet can trigger an outbound order without a CPU ever touching it. Bypass is easier to build and change; full hardware is the choice when every nanosecond and the latency tail matter most. Many shops run both — hardware for the hot path, bypass for everything else.
