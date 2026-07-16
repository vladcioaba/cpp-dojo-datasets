## quiz: What does a successful accept() return?
tags: networking, sockets
track: core

```cpp
int lfd = socket(AF_INET, SOCK_STREAM, 0);
bind(lfd, ...); listen(lfd, 128);
int r = accept(lfd, nullptr, nullptr);
```

- [ ] 0, and the listening socket `lfd` becomes the connection
- [x] A brand-new file descriptor for this one connection; `lfd` keeps listening
- [ ] The client's port number
- [ ] It returns `lfd` itself, now marked connected

> `accept()` pops one completed connection off the listen queue and returns a *new* fd dedicated to that peer. The listening socket is untouched and you keep calling `accept()` on it for further clients. One listening fd, N connection fds — mixing them up is a classic beginner segfault-by-design.

## quiz: A UDP peer sends a 100-byte datagram; you call recvfrom(fd, buf, 40, 0). What happens?
tags: networking, udp
track: core

- [ ] You get 40 bytes now and the remaining 60 on the next call
- [ ] The call fails with EMSGSIZE and nothing is consumed
- [x] You get 40 bytes; the other 60 are discarded forever
- [ ] The kernel blocks until your buffer is large enough

> UDP is datagram-oriented: one `recvfrom` consumes one whole datagram. Whatever doesn't fit in your buffer is silently thrown away (you can detect it with `MSG_TRUNC`). There is no "rest of the message" — unlike TCP, where unread stream bytes stay queued. Always size UDP receive buffers to the largest datagram your protocol allows.

## quiz: A TCP client calls send() twice: "HELLO" then "WORLD". What can the server's recv() return?
tags: networking, tcp, framing
track: core

- [ ] Exactly "HELLO", then exactly "WORLD" — TCP preserves send boundaries
- [ ] "HELLOWORLD" is possible, but a split like "HELLOWO" / "RLD" is not
- [x] Any split of the 10 bytes: "HELLOWORLD", "HEL" then "LOWORLD", etc. — boundaries don't exist
- [ ] It depends on TCP_NODELAY: with Nagle off, boundaries are preserved

> TCP is a byte stream. `send()` boundaries are invisible to the receiver: segments can be coalesced by Nagle, split by MSS, re-chunked by retransmission. Two sends may arrive as one recv, one send as many recvs. Every TCP protocol therefore needs *framing* — length prefixes or delimiters — to rebuild messages. TCP_NODELAY changes timing, not semantics.

## quiz: What is TIME_WAIT actually for?
tags: networking, tcp
track: core

- [ ] It gives the application time to call close() on its end
- [x] It re-ACKs a retransmitted FIN if the last ACK was lost, and lets stale segments die before the port pair is reused
- [ ] It waits for the peer to finish reading buffered data
- [ ] It is a Linux implementation quirk that can safely be disabled

> The active closer holds the connection for 2×MSL for two correctness reasons: if its final ACK is lost the peer will retransmit FIN and something must still exist to answer it; and any delayed duplicate segments from the old connection must expire before a new connection can reuse the same 4-tuple, or they could be injected into the new stream. It's protocol correctness, not a cleanup delay.

## quiz: recv() on a non-blocking socket returns -1 with errno == EWOULDBLOCK. What does that mean?
tags: networking, nonblocking
track: core

- [ ] The connection was reset by the peer
- [ ] The kernel receive buffer overflowed and data was lost
- [x] Nothing to read right now — not an error; wait for readability and retry
- [ ] The socket wasn't set to non-blocking mode correctly

> `EWOULDBLOCK`/`EAGAIN` is the whole point of non-blocking mode: the call that *would have* slept returns immediately and tells you so. The correct reaction is to arm epoll/kqueue for `EPOLLIN` and retry when notified. Treating it as a fatal error (a surprisingly common bug) tears down perfectly healthy connections under light load.

## quiz: An epoll edge-triggered (EPOLLET) server reads exactly 4096 bytes per wakeup, then waits again. A client sends 10000 bytes in one burst. What happens?
tags: networking, epoll
track: core

- [ ] epoll_wait keeps firing until all 10000 bytes are read
- [x] One wakeup, one 4096-byte read — the remaining bytes sit unread and the connection stalls until the client sends more
- [ ] The kernel discards the unread 5904 bytes
- [ ] EPOLLET automatically re-arms after 100ms as a fallback

> Edge-triggered means "notify on the not-ready→ready *transition*". The burst caused one transition, so one notification. Data still buffered does not re-fire the event; only *new* incoming bytes would. The ET contract is to loop reading until `EAGAIN` on every wakeup. Level-triggered mode would keep reporting readiness — that's the forgiving default.

## quiz: On a little-endian x86-64 host, what does htons(0x1234) return?
tags: networking, endianness
track: core

- [ ] 0x1234 — htons is a no-op on x86
- [x] 0x3412
- [ ] 0x4321
- [ ] 0x2143

> Network order is big-endian; a little-endian host must swap the two bytes of a 16-bit value: 0x12,0x34 becomes 0x34,0x12 → `0x3412` when read back as a host integer. (Nibbles within a byte never move — 0x4321 and 0x2143 are the classic distractors.) On a big-endian host `htons` returns its input unchanged, which is why portable code always calls it.

## quiz: Sender and receiver share this struct and use send(fd, &m, sizeof m, 0). Why is this broken as a wire protocol?
tags: networking, serialization
track: core

```cpp
struct Msg {
    uint32_t seq;
    uint16_t flags;
    uint32_t price;
};
Msg m{1, 2, 3};
send(fd, &m, sizeof m, 0);
```

- [ ] It isn't — both sides use the same struct definition, so the bytes match
- [ ] send() cannot take a struct pointer; it only accepts char buffers
- [x] Layout is compiler/ABI-specific: 2 padding bytes hide after `flags` (sizeof is 12, not 10), field byte order is host-endian, and packing can differ across platforms
- [ ] It works for TCP but not UDP because datagrams strip padding

> The compiler aligns `price` to 4 bytes, inserting 2 invisible padding bytes after `flags` — `sizeof(Msg)` is 12, not 10 — and the padding content is indeterminate (an info leak, too). The integers go out little-endian on x86, backwards per network convention. A peer with a different compiler, arch, or packing pragma reads garbage. Serialize field-by-field at defined offsets in network byte order, or use a schema'd format.

## quiz: In listen(fd, 128), what is 128?
tags: networking, sockets
track: core

- [ ] The maximum number of clients the server can ever serve concurrently
- [ ] The number of worker threads the kernel spawns for accept()
- [x] The queue limit for connections completed (or being established) but not yet accept()ed
- [ ] The receive buffer size in KB for each accepted connection

> The backlog caps the kernel's queue of connections waiting for your `accept()` call (on Linux, the fully-established queue; SYN-flood-resistant half-open handling is separate). Once you accept, a connection leaves the queue — so a server that accepts promptly can serve far more than 128 concurrent clients. If the queue is full, new arrivals are dropped or refused, which clients see as timeouts under load.

## quiz: Your service must detect a dead peer within 5 seconds. Why are application-level heartbeats used instead of TCP keepalive?
tags: networking, tcp, heartbeats
track: core

- [ ] TCP keepalive only works on Windows
- [x] Default keepalive probes start after ~2 hours and only prove the remote TCP stack answers — a deadlocked application still ACKs; heartbeats test the application itself, on your schedule
- [ ] Heartbeats consume less bandwidth than keepalive probes
- [ ] TCP keepalive resets the connection's sequence numbers, corrupting data

> `SO_KEEPALIVE` defaults (tcp_keepalive_time = 7200s on Linux) are uselessly slow for failover, and even tuned per-socket, probes are answered by the *kernel* — a hung process, stuck GC, or wedged event loop keeps ACKing while doing no work. An application heartbeat (ping/pong inside the protocol) proves end-to-end liveness at whatever frequency you need, which is why FIX, exchange sessions, and most RPC frameworks build one in. Keepalive still has a role: reaping connections whose peer machine vanished entirely.
