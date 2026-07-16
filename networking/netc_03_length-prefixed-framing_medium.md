## challenge: Length-prefixed message framing
tags: networking, bytes
track: python
lang: python
difficulty: medium

TCP is a byte stream — your message boundaries do not survive the trip, and `recv()` hands you arbitrary chunks: half a message, three and a bit, one byte. The universal fix is a length prefix. Implement both directions:

- `frame(payload)` → `bytes`: a 4-byte big-endian unsigned length, then the payload.
- `class Deframer` with `feed(chunk) -> list[bytes]`: called with each received chunk in order; returns *all complete messages* newly available (possibly `[]`), in order, buffering any partial data internally until later chunks complete it.

Empty payloads are legal (`frame(b'')` is 4 zero bytes → message `b''`). A chunk boundary can fall anywhere — including in the middle of the 4-byte prefix.

hint: Keep one internal `bytes` buffer. On feed: append the chunk, then repeatedly peel messages off the front while enough bytes are buffered.
hint: You can only read the length once 4 bytes are buffered: `struct.unpack('!I', buf[:4])`. Then you need `4 + n` total before the message is complete — otherwise stop and wait for more.
hint: Extract with slicing — `buf[4:4+n]` is the message, `buf[4+n:]` is the remainder. Loop, because one chunk may complete several messages.

```python
# starter
import struct

def frame(payload):
    ...

class Deframer:
    def __init__(self):
        ...

    def feed(self, chunk):
        ...
```

```python
import struct

def frame(payload):
    return struct.pack('!I', len(payload)) + payload

class Deframer:
    def __init__(self):
        self._buf = b''

    def feed(self, chunk):
        self._buf += chunk
        msgs = []
        while len(self._buf) >= 4:
            (n,) = struct.unpack('!I', self._buf[:4])
            if len(self._buf) < 4 + n:
                break
            msgs.append(self._buf[4:4 + n])
            self._buf = self._buf[4 + n:]
        return msgs
```

```python
# harness
#__USER__
def _check():
    assert frame(b'hi') == b'\x00\x00\x00\x02hi'
    assert frame(b'') == b'\x00\x00\x00\x00'
    stream = frame(b'hello') + frame(b'') + frame(b'world!')
    # one giant chunk -> all three at once
    assert Deframer().feed(stream) == [b'hello', b'', b'world!']
    # dripped one byte at a time -> same messages, in order
    d = Deframer()
    got = []
    for i in range(len(stream)):
        got += d.feed(stream[i:i + 1])
    assert got == [b'hello', b'', b'world!'], got
    # split in the middle of the length prefix
    d2 = Deframer()
    assert d2.feed(stream[:2]) == []
    assert d2.feed(stream[2:11]) == [b'hello']
    assert d2.feed(stream[11:]) == [b'', b'world!']
    # leftover partial stays buffered until completed
    d3 = Deframer()
    assert d3.feed(frame(b'abc')[:5]) == []
    assert d3.feed(frame(b'abc')[5:] + frame(b'x')) == [b'abc', b'x']
    print("PASS")

_check()
```

**Editorial:** The deframer is a tiny state machine flattened into one invariant: *buffer everything, then peel complete frames off the front*. The `while` loop matters — one chunk can complete zero, one, or many messages — and both exit conditions matter: fewer than 4 bytes (can't even read the length) or fewer than `4 + n` (message still in flight). This accumulate-and-peel pattern is inside every protocol implementation that sits on TCP: HTTP/2 frames, Kafka, Redis, gRPC, database drivers. A production version would cap `n` (a hostile 4 GB prefix is a memory-exhaustion attack) and use a deque or index rather than re-slicing the buffer, but the logic is exactly this.
