## challenge: Varint encode/decode (protobuf-style)
tags: networking, bytes
track: python
lang: python
difficulty: hard

Protocol Buffers (and WebAssembly, and many storage formats) encode unsigned integers as **base-128 varints**: 7 bits of payload per byte, least-significant group first, with the high bit of each byte as a continuation flag — 1 means "more bytes follow", 0 means "this is the last one". Small numbers cost one byte; 64-bit giants cost ten.

Implement:
- `varint_encode(n)` → `bytes` for a non-negative int (arbitrary size).
- `varint_decode(data, offset=0)` → `(value, new_offset)`: decode one varint starting at `offset`, returning the value and the offset just past it — so consecutive varints in one buffer decode by threading the offset.

Worked example: 300 is `0b100101100` → groups of 7 from the low end: `0101100`, `0000010` → emit low group first with continuation bit: `0xAC 0x02`.

hint: Encode loop: take `n & 0x7F`, shift `n >>= 7`; if anything remains, emit the group with `| 0x80` and continue, else emit it bare and stop. Zero must still emit one byte, b'\x00'.
hint: Decode loop: for each byte, OR `(b & 0x7F) << shift` into the result with shift going 0, 7, 14, ...; a clear high bit ends the varint.
hint: Little-endian group order means the FIRST byte holds the LOWEST 7 bits — the decoder's shift grows as bytes arrive; the encoder never needs to know the length in advance.

```python
# starter
def varint_encode(n):
    ...

def varint_decode(data, offset=0):
    ...
```

```python
def varint_encode(n):
    out = bytearray()
    while True:
        b = n & 0x7F
        n >>= 7
        if n:
            out.append(b | 0x80)
        else:
            out.append(b)
            return bytes(out)

def varint_decode(data, offset=0):
    result = 0
    shift = 0
    while True:
        b = data[offset]
        offset += 1
        result |= (b & 0x7F) << shift
        if not (b & 0x80):
            return result, offset
        shift += 7
```

```python
# harness
#__USER__
def _check():
    assert varint_encode(0) == b'\x00'
    assert varint_encode(1) == b'\x01'
    assert varint_encode(127) == b'\x7f'
    assert varint_encode(128) == b'\x80\x01'
    assert varint_encode(300) == b'\xac\x02'          # the protobuf docs example
    assert varint_encode(16384) == b'\x80\x80\x01'
    assert varint_decode(b'\xac\x02') == (300, 2)
    assert varint_decode(b'\xff\xff\xff\xff\x0f') == (2**32 - 1, 5)
    # offset threading: decode consecutive varints out of one buffer
    buf = varint_encode(5) + varint_encode(1000) + varint_encode(0)
    v1, off = varint_decode(buf)
    v2, off = varint_decode(buf, off)
    v3, off = varint_decode(buf, off)
    assert (v1, v2, v3) == (5, 1000, 0) and off == len(buf)
    # round-trip a spread of values, including group boundaries
    for n in [0, 1, 127, 128, 255, 300, 2**14 - 1, 2**14, 2**21, 2**35 + 7, 2**63]:
        enc = varint_encode(n)
        assert varint_decode(enc) == (n, len(enc)), n
    print("PASS")

_check()
```

**Editorial:** Each byte carries a 7-bit group plus a continuation flag, lowest group first. The encoder peels 7 bits at a time until nothing remains (with the "emit at least one byte" edge case for 0); the decoder rebuilds by shifting each group into place — `result |= (b & 0x7F) << shift` with shift stepping by 7 — until it sees a clear high bit. The boundary values are where bugs live: 127 (one byte, `0x7F`) vs 128 (first two-byte value, `0x80 0x01`), and every power of `2**7k`. Returning `(value, new_offset)` instead of consuming a stream is the standard zero-copy decoding shape — protobuf fields are `(tag varint, value)` pairs threaded through a buffer exactly like this. Signed integers add ZigZag on top (`(n << 1) ^ (n >> 63)`) so small negatives stay small; a hardened decoder also caps the byte count (10 for 64-bit) to reject malicious inputs.
