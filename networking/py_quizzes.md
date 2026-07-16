## quiz: sock.send(payload) returned 5, but len(payload) is 100. What happened?
tags: networking, sockets
track: python

- [ ] Impossible — send() always transmits the whole buffer or raises
- [x] Only the first 5 bytes were queued; sending the remaining 95 is your job (or use sendall)
- [ ] The other 95 bytes were sent out-of-band and will arrive separately
- [ ] The connection dropped after 5 bytes; the socket is now closed

> `send()` queues as much as fits in the kernel send buffer and returns the count — a short send is normal under backpressure. Code that ignores the return value silently truncates messages. `sendall()` is the retry loop done for you: it returns only when everything is queued, or raises. This mirrors C's `send()`; the trap survives translation.

## quiz: What does struct.pack('>H', 258) produce?
tags: networking, bytes, struct
track: python

- [ ] b'\x02\x01'
- [x] b'\x01\x02'
- [ ] b'\x00\x02\x00\x01'
- [ ] b'258'

> `>H` is a big-endian unsigned 16-bit integer. 258 = 0x0102, and big-endian writes the most significant byte first: `b'\x01\x02'`. Little-endian (`<H`) would give `b'\x02\x01'`. Two bytes, not four (`H` is 16-bit; `I` is 32), and never ASCII digits — `struct` is binary, not `str(258)`.

## quiz: You call sock.recv(4096) on a TCP socket. The peer sent 10 bytes. What does recv return?
tags: networking, sockets
track: python

- [ ] It blocks until 4096 bytes have accumulated
- [ ] Exactly 4096 bytes, zero-padded after the first 10
- [x] Up to 4096 bytes — here the 10 available; b'' would mean the peer closed
- [ ] It raises BufferError because the peer sent less than requested

> The argument to `recv` is a *maximum*, not a contract: it returns whatever is in the receive buffer right now, at most 4096 bytes. Ten bytes available → ten bytes returned. The special case is `b''` (empty bytes), which means the peer closed the connection cleanly. Any "read exactly k bytes" protocol logic must loop and accumulate — or use asyncio's `readexactly`.

## quiz: Inside a coroutine, execution hits `await something_not_ready`. Who gets control?
tags: networking, asyncio
track: python

- [ ] The operating system scheduler, which parks the thread
- [ ] The next line of the same coroutine, which runs speculatively
- [x] The event loop, which resumes other coroutines until this one's result is ready
- [ ] A worker thread from asyncio's internal thread pool that completes the await

> `await` on a pending awaitable suspends the coroutine and yields control back to the event loop — the same single thread then runs whatever other coroutine is ready. Nothing is preempted and no thread pool is involved (unless you explicitly use `asyncio.to_thread`). That's the whole model: cooperative multitasking with suspension points spelled `await`.

## quiz: What does `results = await asyncio.gather(fetch(a), fetch(b), fetch(c))` do?
tags: networking, asyncio
track: python

- [ ] Runs the three coroutines strictly one after another, left to right
- [ ] Returns results in completion order, fastest first
- [x] Runs all three concurrently and returns their results in argument order
- [ ] Returns the result of whichever finishes first and cancels the rest

> `gather` schedules all awaitables concurrently on the loop and completes when all are done, with results positionally matching the arguments — completion order is irrelevant to the result list. "Fastest first" describes `asyncio.as_completed`; "first one wins" is `asyncio.wait(..., return_when=FIRST_COMPLETED)`. By default one exception propagates and cancels nothing else unless you handle it (`return_exceptions=True` collects errors as values).

## quiz: What's the difference between sock.close() and sock.shutdown(socket.SHUT_WR)?
tags: networking, sockets
track: python

- [ ] None — shutdown is an alias kept for C compatibility
- [x] shutdown(SHUT_WR) sends FIN — "no more data from me" — while you can still recv; close() just releases this handle's reference to the socket
- [ ] close() sends FIN immediately; shutdown only clears Python's buffers
- [ ] shutdown works only on listening sockets, close only on connections

> `shutdown(SHUT_WR)` performs a half-close: your FIN goes out, the peer's `recv` returns `b''`, yet you can keep reading their remaining data — exactly what a client that has finished sending a request wants. `close()` only drops this file object's reference; the underlying connection tears down when the last reference (think `dup`'d fds, forked children) disappears. The classic use: shutdown write, drain the response, then close.

## quiz: Inside an asyncio server you call a plain blocking srv_sock.accept(). Why is this a disaster?
tags: networking, asyncio
track: python

- [ ] accept() is thread-unsafe and corrupts the event loop's internal state
- [ ] Blocking calls raise RuntimeError when a loop is running in the thread
- [x] The whole event loop freezes: every other coroutine — heartbeats, timeouts, existing clients — stops until a new client happens to connect
- [ ] Nothing serious: asyncio detects the block and moves other tasks to a thread

> The loop is one thread running one thing at a time; coroutines only interleave at `await` points. A blocking `accept()` occupies that thread indefinitely, so *all* concurrent work stalls — connected clients time out because their coroutines never get scheduled. asyncio does not detect or offload this (it can only *warn* in debug mode). Use `await loop.sock_accept(sock)`, `asyncio.start_server`, or push truly blocking work through `asyncio.to_thread`.

## quiz: What does sock.send("hello") do on a connected TCP socket?
tags: networking, bytes
track: python

- [ ] Sends the 5 ASCII bytes — Python encodes str automatically
- [ ] Sends it UTF-8 encoded with a BOM prefix
- [x] Raises TypeError: a bytes-like object is required, not 'str'
- [ ] Works in Python 2 and 3 identically

> The wire carries bytes; Python 3's `str` is a sequence of Unicode code points with no defined byte representation until *you* pick an encoding. So sockets accept only bytes-like objects and `send("hello")` raises `TypeError: a bytes-like object is required, not 'str'`. Encode explicitly at the boundary — `sock.sendall("hello".encode("utf-8"))` — and `.decode()` on the way in. (Python 2's blurring of str/bytes is precisely the bug class Python 3 eliminated.)
