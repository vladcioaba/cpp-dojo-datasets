## fact: The sockets API is five calls
tags: networking, sockets
track: core

Every TCP server ever written is the same skeleton: `socket()` makes an endpoint (a file descriptor), `bind()` attaches it to an address and port, `listen()` turns it into a passive socket with a queue of incoming connections, and `accept()` pops one completed connection off that queue — returning a **new** fd for the conversation. Clients skip bind/listen/accept and just `connect()`.

The listening fd and the connection fds are different objects with different jobs. You `accept()` on one, `read()`/`write()` on the others.

```cpp
int lfd = socket(AF_INET, SOCK_STREAM, 0);
sockaddr_in addr{};
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = htonl(INADDR_ANY);
addr.sin_port = htons(8080);
bind(lfd, reinterpret_cast<sockaddr*>(&addr), sizeof addr);
listen(lfd, 128);                    // backlog of pending connections
int cfd = accept(lfd, nullptr, nullptr);  // NEW fd per client
// read()/write() on cfd; keep accept()ing on lfd
```

## fact: TCP vs UDP — a byte stream vs stamped postcards
tags: networking, tcp, udp
track: core

TCP (`SOCK_STREAM`) gives you a connection-oriented, reliable, ordered **byte stream**: bytes arrive exactly once, in order, or the connection errors out. The price is handshakes, retransmission delays, and head-of-line blocking — one lost segment stalls everything behind it.

UDP (`SOCK_DGRAM`) gives you connectionless **datagrams**: each `sendto()` becomes one packet that may arrive out of order, duplicated, or not at all — but message boundaries are preserved, and there is no connection state or retransmission latency. That is why market data feeds are UDP multicast and order entry is TCP: the feed tolerates a gap (you re-request or recover), but an order must not be lost.

Rule of thumb: TCP when you need everything and can wait; UDP when you need it *now* and can handle loss yourself.

## fact: The TCP handshake, teardown, and why TIME_WAIT exists
tags: networking, tcp
track: core

Setup is the three-way handshake: SYN → SYN-ACK → ACK. Both sides pick random initial sequence numbers and agree on options (window scale, MSS, SACK). Only after this does `connect()` return and `accept()` have something to deliver.

Teardown is four-way: each direction closes independently with its own FIN/ACK pair (that's what makes half-close, `shutdown(fd, SHUT_WR)`, possible). The side that closes **first** enters `TIME_WAIT` and lingers for 2×MSL (typically 60s on Linux). Two reasons: (1) if its last ACK is lost, the peer retransmits FIN and someone must still be there to re-ACK it; (2) it keeps the port 4-tuple quarantined so delayed old segments can't be mistaken for data on a fresh connection reusing the same tuple.

TIME_WAIT is not a bug or a leak — it is correctness. Servers that churn thousands of short connections manage it (connection pooling, `SO_REUSEADDR`, letting the *client* close first) rather than disabling it.

## fact: Blocking vs non-blocking sockets and EAGAIN
tags: networking, nonblocking
track: core

By default sockets **block**: `read()` sleeps until data arrives, `write()` sleeps until the kernel buffer has room, `accept()` sleeps until a connection lands. Fine for one connection per thread; fatal for an event loop.

Set `O_NONBLOCK` and those calls return immediately. If the operation *would have* blocked, they fail with `errno == EAGAIN` (a.k.a. `EWOULDBLOCK` — same value on Linux). That is not an error; it's the kernel saying "nothing for you right now, come back when I tell you." Non-blocking I/O only makes sense paired with a readiness notifier (epoll/kqueue) that tells you *when* to come back.

```cpp
int flags = fcntl(fd, F_GETFL, 0);
fcntl(fd, F_SETFL, flags | O_NONBLOCK);

ssize_t n = read(fd, buf, sizeof buf);
if (n < 0 && (errno == EAGAIN || errno == EWOULDBLOCK)) {
    // not an error: no data yet — wait for readability, then retry
}
```

Also remember: a non-blocking `write()` can accept *part* of your buffer. Track how much was taken and resubmit the rest when writable.

## fact: select → poll → epoll/kqueue → io_uring
tags: networking, multiplexing
track: core

The evolution of "watch N sockets at once" is an evolution in *who does the scanning*:

- **select** (1983): pass a bitmask of fds, kernel scans them all, you scan the result. O(n) per call both ways, hard `FD_SETSIZE` cap (typically 1024), and the fd_set is destroyed each call so you rebuild it every time.
- **poll**: same O(n) model but an array of `pollfd` instead of a bitmask — no 1024 cap, no rebuild. Still linear.
- **epoll** (Linux) / **kqueue** (BSD/macOS): register interest *once*; the kernel maintains the interest list and hands you only the fds that are ready. `epoll_wait` is O(ready), not O(watched). This is what made 100k+ concurrent connections practical and what every event loop (libuv, asio, nginx) sits on.
- **io_uring**: goes past *readiness* to *completion*. You submit operations (read, write, accept...) into a shared ring buffer and reap completions from another — often without a syscall in the hot path. You no longer say "tell me when I can read"; you say "read it and tell me when it's done."

Interview sound bite: select/poll are stateless-per-call and linear; epoll/kqueue are stateful readiness; io_uring is async completion.

## fact: Edge-triggered vs level-triggered readiness
tags: networking, epoll
track: core

**Level-triggered** (the default for epoll, and the only mode of select/poll): as long as the socket *has* readable data, every wait call reports it readable. Forgiving — read some bytes, leave the rest, you'll be told again.

**Edge-triggered** (`EPOLLET`): you are notified only on the *transition* from not-ready to ready, i.e., when new bytes arrive. If you read only part of the buffer and go back to waiting, no new notification comes for the remainder — the data just sits there and the connection hangs.

The ET contract is therefore: on every wakeup, loop `read()`/`accept()` until you get `EAGAIN`. ET's payoff is fewer wakeups and no re-scanning of half-drained sockets under load, which is why high-performance servers use it — but only with non-blocking fds and drain-until-EAGAIN discipline.

```cpp
epoll_event ev{};
ev.events = EPOLLIN | EPOLLET;   // edge-triggered: drain or hang
ev.data.fd = cfd;
epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
// on wakeup: while (read(cfd, ...) > 0) {} until EAGAIN
```

## fact: Nagle's algorithm and TCP_NODELAY
tags: networking, tcp, latency, hft
track: core

Nagle's algorithm batches small writes: if there is unacknowledged data in flight, the kernel holds new small segments and coalesces them until an ACK returns. Great for telnet in 1984 — it stops a torrent of 1-byte packets. Terrible for latency-sensitive request/response: your 40-byte order can sit in the kernel waiting for an ACK, and it interacts pathologically with **delayed ACK** on the other side (each waiting for the other → bursts of ~40ms stalls).

`TCP_NODELAY` disables Nagle: every write goes out as soon as the window allows. In HFT, trading gateways set it unconditionally — a predictable extra small packet beats an unpredictable multi-millisecond stall every time. Any low-latency RPC (Redis, gRPC, game servers) does the same.

```cpp
int one = 1;
setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof one);
```

Complementary trick: coalesce in *your* code (build the full message, one `write()`) so you don't need Nagle to fix a chatty writer.

## fact: SO_REUSEADDR vs SO_REUSEPORT
tags: networking, sockets
track: core

`SO_REUSEADDR` answers a restart problem: after a server dies, its port lingers in `TIME_WAIT`, and a plain `bind()` fails with `EADDRINUSE` until it expires. Setting `SO_REUSEADDR` before `bind()` lets you rebind while old connections drain. Practically every server sets it; forgetting it is the classic "can't restart my server for a minute" bug.

`SO_REUSEPORT` (Linux 3.9+) is a different beast: it lets **multiple live sockets** — typically one per worker process/thread — bind the *same* address and port, and the kernel load-balances incoming connections (or UDP datagrams) across them by 4-tuple hash. That removes the single shared accept queue and the thundering-herd/lock contention on it. nginx and modern multi-worker servers use it for linear accept scaling.

```cpp
int one = 1;
setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &one, sizeof one); // rebind past TIME_WAIT
setsockopt(lfd, SOL_SOCKET, SO_REUSEPORT, &one, sizeof one); // N workers, same port
```

## fact: Endianness on the wire — htons and friends
tags: networking, endianness
track: core

Network byte order is **big-endian**: most significant byte first. x86 and ARM (as commonly run) are little-endian. Every multi-byte integer that touches the wire — ports and addresses in `sockaddr_in`, lengths and IDs in your own protocol headers — must be converted, or two little-endian hosts will happily agree on the wrong numbers the moment a big-endian peer, a spec-conformant implementation, or a packet capture enters the picture.

The four converters: `htons`/`htonl` (host-to-network, 16/32-bit) and `ntohs`/`ntohl` (network-to-host). On a little-endian machine they byte-swap — `htons(0x1234) == 0x3412` — and on a big-endian machine they compile to nothing, which is exactly why you always write them: the code is then correct *everywhere*.

```cpp
addr.sin_port = htons(8080);          // 0x1F90 -> wire order
uint32_t len_net;
memcpy(&len_net, buf, 4);             // 4 wire bytes
uint32_t len = ntohl(len_net);        // back to host order
```

C++20/23 add `std::endian` and `std::byteswap` for the same job in pure C++.

## fact: Never memcpy a struct onto the wire
tags: networking, serialization
track: core

`send(fd, &msg, sizeof msg, 0)` looks like free serialization. It isn't — it ships your compiler's private memory layout as if it were a protocol. Three things break it: **padding** (the compiler inserts invisible bytes to align members — and their content is indeterminate, so you may also leak stack garbage), **endianness** (little-endian ints are backwards on the wire), and **ABI drift** (a different compiler, architecture, or packing flag reads different offsets).

```cpp
struct Msg {
    uint32_t seq;    // offset 0
    uint16_t flags;  // offset 4
    uint32_t price;  // offset 8 — after 2 bytes of padding!
};
static_assert(sizeof(Msg) == 12);  // not 10: 2 padding bytes hide inside
```

The fix is explicit serialization: write each field at a defined offset in a defined byte order (`htonl` per field into a byte buffer), or use a schema'd format (protobuf, FlatBuffers, SBE). Exchange protocols that *do* specify C-like layouts pin everything down — fixed offsets, specified endianness, `#pragma pack` — and even then you serialize field-by-field at the boundary, not by trusting `sizeof`.

## fact: Zero-copy — sendfile and MSG_ZEROCOPY
tags: networking, zero-copy
track: core

A naive file-to-socket loop copies every byte four times: disk → page cache → your buffer (`read`), your buffer → socket buffer (`send`), plus two user/kernel crossings per chunk. For a static-file server or proxy, the user-space detour adds nothing.

`sendfile(out_fd, in_fd, ...)` moves data page-cache-to-socket entirely inside the kernel — no user-space copy, no double syscall per chunk. It's why nginx serves static content at line rate. `splice()` generalizes it through pipes.

`MSG_ZEROCOPY` (Linux 4.14+) attacks the *send-side* copy for ordinary in-memory buffers: `send(fd, buf, n, MSG_ZEROCOPY)` pins your pages and the NIC reads them directly via DMA; the kernel posts a completion on the error queue when it's done — until then you must not touch `buf`. That deferred-completion bookkeeping has real overhead, so it only wins for large transfers (rule of thumb: ~10 KB+); for small writes the plain copy is faster.

The theme: copies and syscalls, not bandwidth, are the tax. Zero-copy techniques remove the memcpy; io_uring removes the syscalls; kernel bypass removes the kernel.

## fact: Kernel bypass — DPDK and ef_vi
tags: networking, kernel-bypass, hft
track: core

The kernel network stack costs microseconds: interrupt, context switch, sk_buff allocation, protocol demux, socket queue, wakeup. A syscall-based echo hovers around tens of microseconds round trip. HFT budgets are sub-microsecond wire-to-wire — so the kernel has to go.

Kernel bypass maps the NIC's RX/TX rings straight into user space and **polls** them from a pinned, isolated core — no interrupts, no syscalls, no copies:

- **DPDK** (Intel-originated, portable): takes over the whole NIC via a userspace poll-mode driver, hugepage-backed buffer pools, burst APIs. You bring (or buy) your own TCP/IP stack.
- **ef_vi / Onload** (Solarflare/Xilinx/AMD NICs — the HFT house standard): `ef_vi` is the raw layer-2 API for hand-rolled paths; Onload is a drop-in userspace TCP/UDP stack behind the ordinary sockets API — bypass without rewriting the app.

Costs, and why interviews probe it: a burned 100%-CPU polling core per queue, no kernel firewalling/netfilter/tcpdump on that traffic, and you own reliability. Typical shape: market data and order path on bypass; everything else through the kernel. Further down the rabbit hole, the same logic moves the strategy itself into FPGA/NIC hardware.
