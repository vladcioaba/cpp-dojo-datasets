## fact: The socket module is the C API with a Python accent
tags: networking, sockets
track: python

`import socket` exposes Berkeley sockets almost 1:1: `socket.socket(socket.AF_INET, socket.SOCK_STREAM)` is a TCP/IPv4 endpoint, `SOCK_DGRAM` is UDP, `AF_INET6` is IPv6. The C dance — bind/listen/accept for servers, connect for clients — is the same, just with tuples `(host, port)` instead of `sockaddr` structs, and exceptions (`OSError`, `ConnectionRefusedError`) instead of -1/errno.

```python
import socket

srv = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
srv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
srv.bind(("0.0.0.0", 8080))
srv.listen(128)
conn, peer = srv.accept()      # NEW socket per client + peer address tuple
data = conn.recv(4096)         # bytes, up to 4096 — maybe fewer
```

Knowing this layer pays off even if you live in asyncio or requests: every abstraction above it leaks its semantics — partial reads, byte streams, connection resets.

## fact: Context-managed sockets, and sendall vs send
tags: networking, sockets
track: python

Sockets are resources — use `with` so they close on every exit path, exceptions included (Python's RAII). And learn the one trap that bites everyone: `send()` is **not obligated to send everything**. It returns the number of bytes actually queued, which can be less than you gave it, and it's on you to loop. `sendall()` is that loop, provided by the library — it either sends the whole buffer or raises.

```python
import socket

with socket.create_connection(("example.com", 80), timeout=5) as s:
    n = s.send(payload)        # may send only a prefix! returns count
    s.sendall(payload)         # sends everything or raises — use this
```

`socket.create_connection()` is the friendly client constructor: resolves the host, tries each address, applies the timeout. Symmetrically on the read side, `recv(n)` returns *up to* n bytes — an empty `b""` means the peer closed. Any "read exactly k bytes" logic must loop.

## fact: struct.pack and unpack are your binary protocol toolkit
tags: networking, bytes, struct
track: python

Binary protocols define fields at byte offsets in a fixed byte order. The `struct` module converts between those bytes and Python values using format strings: `!` means network order (big-endian) — start every wire format with it; `B`/`H`/`I`/`Q` are unsigned 8/16/32/64-bit ints; `s` with a count is fixed-length raw bytes.

```python
import struct

struct.pack('!H', 80)                    # b'\x00\x50'   — port as 16-bit BE
struct.pack('!IH', 1, 2)                 # b'\x00\x00\x00\x01\x00\x02'
struct.unpack('!IH', b'\x00\x00\x00\x01\x00\x02')  # (1, 2)
struct.calcsize('!IHB')                  # 7 — no padding with ! (unlike native '@')
```

Two habits: always write the endianness prefix (native mode `@` inserts C-style padding and host byte order — exactly what you don't want on the wire), and use `struct.calcsize(fmt)` instead of hand-counting header sizes. `unpack` demands an exactly-sized buffer; use `unpack_from(fmt, buf, offset)` to walk through a larger one.

## fact: async/await — cooperative multitasking on one thread
tags: networking, asyncio
track: python

The asyncio mental model: **one thread, one event loop, many paused functions.** A coroutine (`async def`) runs until it hits an `await` on something not ready — then it *suspends*, handing control back to the event loop, which runs whichever other coroutine's I/O just completed. Nothing runs in parallel; things *interleave* at await points.

Two consequences follow. First, `await` marks every place your function can be paused — between awaits you cannot be preempted (no locks needed for plain data). Second, any **blocking** call — `time.sleep`, `requests.get`, a blocking `sock.recv` — freezes the *entire loop*: no other coroutine runs until it returns. That's the cardinal sin. Use the async equivalents (`asyncio.sleep`, async clients) or push blocking work to a thread with `await asyncio.to_thread(fn)`.

```python
import asyncio

async def fetch(host):
    await asyncio.sleep(0.1)          # suspends HERE; loop runs others
    return host

async def main():
    # gather runs them concurrently; results come back in argument order
    results = await asyncio.gather(fetch("a"), fetch("b"), fetch("c"))

asyncio.run(main())                    # creates the loop, runs main, closes it
```

## fact: asyncio streams — sockets with await
tags: networking, asyncio
track: python

`asyncio.open_connection()` and `asyncio.start_server()` are the high-level async socket API. They hand you a `StreamReader`/`StreamWriter` pair wrapping a non-blocking socket registered with the loop: `await reader.read(n)` suspends instead of blocking; `readexactly(n)` and `readline()` do the accumulate-a-full-message loop for you (a length-prefixed protocol is `readexactly(4)` then `readexactly(length)`).

```python
import asyncio

async def handle(reader, writer):          # one running instance per client
    data = await reader.readline()
    writer.write(data.upper())             # buffers, doesn't send by itself
    await writer.drain()                   # backpressure: wait for the buffer to flush
    writer.close()
    await writer.wait_closed()

async def main():
    server = await asyncio.start_server(handle, "127.0.0.1", 8888)
    async with server:
        await server.serve_forever()
```

Note the split: `write()` is synchronous buffering, `await drain()` applies backpressure when the peer reads slowly. Ten thousand concurrent clients are ten thousand paused `handle` coroutines — no threads, one loop.

## fact: HTTP clients — urllib, http.client, and the requests shape
tags: networking, http
track: python

Under every HTTP library is the same socket work: connect (TLS handshake for https), send `METHOD path HTTP/1.1` plus headers, parse the status line and headers back, read the body per `Content-Length` or chunked encoding, maybe keep the connection alive for reuse.

The stdlib gives you two layers: `http.client` is that protocol made explicit (`HTTPSConnection`, `request()`, `getresponse()`); `urllib.request.urlopen()` adds URL parsing and redirect handling. The third-party `requests` (and its async sibling `httpx`) defines the API shape everyone copies:

```python
import urllib.request, json

with urllib.request.urlopen("https://api.example.com/data", timeout=10) as r:
    payload = json.loads(r.read().decode("utf-8"))

# the requests-shaped equivalent every library imitates:
# r = requests.get(url, timeout=10); r.raise_for_status(); payload = r.json()
```

The concepts that transfer across all of them: **sessions/connection pooling** (reusing TCP+TLS connections dwarfs any other client-side optimization), **always set a timeout** (the default is often "forever"), and status handling is on you (`urlopen` raises on 4xx/5xx; `requests` doesn't until `raise_for_status()`).

## fact: The GIL doesn't matter for IO-bound work
tags: networking, concurrency, gil
track: python

The GIL lets only one thread execute Python bytecode at a time — so threads don't speed up CPU-bound pure-Python code. But every blocking I/O call in CPython (`sock.recv`, `sock.send`, file reads, `time.sleep`) **releases the GIL** while waiting in the kernel. Fifty threads waiting on fifty sockets are all genuinely waiting in parallel; the GIL only serializes the Python-level processing between waits, and IO-bound work is mostly waits.

So for network concurrency you have two honest options: **threads** (simple, pre-emptive, fine up to hundreds of connections, each costing a stack and scheduler overhead) and **asyncio** (single-threaded cooperative, cheap enough for tens of thousands of connections, but one blocking call stalls everything). Both are legitimate; neither is throttled by the GIL for I/O.

CPU-bound work is different: use `multiprocessing`/`ProcessPoolExecutor` (separate interpreters, separate GILs) or a C extension that releases the GIL (numpy). Interview one-liner: *GIL blocks parallel Python execution, not parallel waiting.*

## fact: selectors — the epoll abstraction under asyncio
tags: networking, multiplexing
track: python

The `selectors` module is Python's portable readiness API: `DefaultSelector` picks the best mechanism for the platform — epoll on Linux, kqueue on macOS/BSD, and select as the fallback — behind one interface. `register(fd, events, data)` declares interest; `select(timeout)` returns the fds that are ready right now. This is exactly the layer asyncio's default event loop is built on, so writing one loop by hand demystifies asyncio permanently.

```python
import selectors, socket

sel = selectors.DefaultSelector()
srv = socket.socket(); srv.bind(("0.0.0.0", 8080)); srv.listen()
srv.setblocking(False)
sel.register(srv, selectors.EVENT_READ, data="accept")

while True:
    for key, events in sel.select(timeout=1.0):
        if key.data == "accept":
            conn, _ = key.fileobj.accept()
            conn.setblocking(False)
            sel.register(conn, selectors.EVENT_READ, data="client")
        else:
            data = key.fileobj.recv(4096)
            if not data:                       # peer closed
                sel.unregister(key.fileobj); key.fileobj.close()
```

The pattern — non-blocking fds, a registry of interests, a dispatch loop over ready events — *is* the event loop. asyncio adds coroutines, callbacks, and scheduling sugar on top of exactly this.
