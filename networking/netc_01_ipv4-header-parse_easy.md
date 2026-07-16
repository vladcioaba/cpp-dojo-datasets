## challenge: Parse an IPv4 header
tags: networking, bytes
track: python
lang: python
difficulty: easy

You receive raw bytes as they came off the wire: an IPv4 header (possibly with options), then payload. Write `parse_ipv4(data)` returning a dict with `'version'` (int), `'ihl'` (header length **in bytes**, i.e. the IHL field × 4), `'ttl'` (int), and `'src'`/`'dst'` as dotted-quad strings like `'192.168.0.1'`.

The fixed 20-byte header layout (all multi-byte fields big-endian): byte 0 packs version in the high nibble and IHL (in 32-bit words) in the low nibble; then TOS (1B), total length (2B), identification (2B), flags+fragment offset (2B), TTL (1B), protocol (1B), header checksum (2B), source address (4B), destination address (4B).

Example: `bytes.fromhex('450000541c4640003906b1e6ac100a63080808ff')` → version 4, ihl 20, ttl 57, src `'172.16.10.99'`, dst `'8.8.8.255'`.

hint: One struct format string unpacks the whole fixed header: `'!BBHHHBBH4s4s'` over `data[:20]`. `!` is network (big-endian) order.
hint: Version and IHL share a byte: `b >> 4` is the high nibble, `b & 0x0F` the low. IHL counts 32-bit words — multiply by 4 for bytes.
hint: A `4s` field unpacks as 4 raw bytes; iterating over `bytes` yields ints, so `'.'.join(str(b) for b in src)` builds the dotted quad.

```python
# starter
import struct

def parse_ipv4(data):
    ...
```

```python
import struct

def parse_ipv4(data):
    vihl, tos, total_len, ident, frag, ttl, proto, csum, src, dst = \
        struct.unpack('!BBHHHBBH4s4s', data[:20])
    return {
        'version': vihl >> 4,
        'ihl': (vihl & 0x0F) * 4,
        'ttl': ttl,
        'src': '.'.join(str(b) for b in src),
        'dst': '.'.join(str(b) for b in dst),
    }
```

```python
# harness
#__USER__
import struct as _struct

def _mk(version, ihl_words, ttl, src, dst):
    src_b = bytes(int(p) for p in src.split('.'))
    dst_b = bytes(int(p) for p in dst.split('.'))
    hdr = _struct.pack('!BBHHHBBH4s4s', (version << 4) | ihl_words,
                       0, 20, 0x1c46, 0x4000, ttl, 6, 0xb1e6, src_b, dst_b)
    return hdr + b'\x00' * (ihl_words * 4 - 20)

def _check():
    h = _mk(4, 5, 64, '192.168.0.1', '10.0.0.42')
    p = parse_ipv4(h)
    assert p['version'] == 4, p
    assert p['ihl'] == 20, p
    assert p['ttl'] == 64, p
    assert p['src'] == '192.168.0.1', p
    assert p['dst'] == '10.0.0.42', p
    # header with options (IHL = 6 -> 24 bytes) and trailing payload
    h2 = _mk(4, 6, 1, '10.1.2.3', '255.255.255.255') + b'payload-bytes'
    p2 = parse_ipv4(h2)
    assert p2['ihl'] == 24 and p2['ttl'] == 1, p2
    assert p2['src'] == '10.1.2.3' and p2['dst'] == '255.255.255.255', p2
    # a hand-built raw capture
    raw = bytes.fromhex('450000541c4640003906b1e6ac100a63080808ff')
    p3 = parse_ipv4(raw)
    assert p3['version'] == 4 and p3['ihl'] == 20 and p3['ttl'] == 0x39, p3
    assert p3['src'] == '172.16.10.99' and p3['dst'] == '8.8.8.255', p3
    print("PASS")

_check()
```

**Editorial:** One `struct.unpack('!BBHHHBBH4s4s', data[:20])` call maps the whole fixed header; the format string *is* the protocol spec, field for field. The only bit-fiddling is splitting the shared version/IHL byte with a shift and a mask, and remembering IHL is in 32-bit words (so ×4 for bytes — headers with options are longer than 20). Addresses come out as `4s` raw bytes and become dotted quads by joining the byte values. Real parsers do exactly this, plus validating version, checksum, and `ihl >= 20`.
