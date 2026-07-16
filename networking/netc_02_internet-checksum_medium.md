## challenge: Internet checksum (RFC 1071)
tags: networking, bytes
track: python
lang: python
difficulty: medium

Implement the checksum used by IPv4, ICMP, UDP and TCP. Write `inet_checksum(data)` → int in [0, 0xFFFF]: treat `data` as a sequence of 16-bit **big-endian** words (if the length is odd, pad the final byte with a zero on the right), add them all up, fold any carry above bit 15 back into the low 16 bits until none remains, and return the one's complement of the result, masked to 16 bits.

The neat property: a receiver that runs the same sum over a header *whose checksum field is filled in* gets 0 for an intact packet — that's how verification works in practice.

Example: for the 20-byte IPv4 header `4500 0073 0000 4000 4011 0000 c0a8 0001 c0a8 00c7` (checksum field zeroed), the checksum is `0xB861`.

hint: Walk the bytes two at a time: `word = (data[i] << 8) | data[i+1]`. Append a single zero byte first when `len(data)` is odd.
hint: Python ints don't overflow, so the carries pile up above bit 15. Fold with `total = (total & 0xFFFF) + (total >> 16)` in a loop — one fold can produce a new carry.
hint: Python's `~total` is negative (infinite-precision two's complement); `~total & 0xFFFF` gives the 16-bit one's complement you want.

```python
# starter
def inet_checksum(data):
    ...
```

```python
def inet_checksum(data):
    if len(data) % 2:
        data += b'\x00'
    total = 0
    for i in range(0, len(data), 2):
        total += (data[i] << 8) | data[i + 1]
    while total >> 16:
        total = (total & 0xFFFF) + (total >> 16)
    return ~total & 0xFFFF
```

```python
# harness
#__USER__
def _check():
    # classic IPv4 header example: checksum field (zeroed here) must come out 0xB861
    hdr = bytes.fromhex('45000073000040004011' + '0000' + 'c0a80001c0a800c7')
    assert inet_checksum(hdr) == 0xB861, hex(inet_checksum(hdr))
    # verifying a received header: sum over the full header (checksum in place) gives 0
    full = bytes.fromhex('45000073000040004011' + 'b861' + 'c0a80001c0a800c7')
    assert inet_checksum(full) == 0, hex(inet_checksum(full))
    assert inet_checksum(b'') == 0xFFFF
    assert inet_checksum(b'\x00\x00') == 0xFFFF
    assert inet_checksum(b'\xff\xff') == 0
    # odd length: trailing byte is padded with a zero on the right
    assert inet_checksum(b'\x01\x02\x03') == inet_checksum(b'\x01\x02\x03\x00')
    # carry folding actually happens
    assert inet_checksum(b'\xff\xff\xff\xff\x00\x01') == 0xFFFE, hex(inet_checksum(b'\xff\xff\xff\xff\x00\x01'))
    print("PASS")

_check()
```

**Editorial:** The algorithm is "one's complement sum of 16-bit words, then complement". The two details that fail interviews: **carry folding** — in one's complement arithmetic a carry out of bit 15 wraps around and is added back in (`(total & 0xFFFF) + (total >> 16)`, looped, since a fold can itself carry) — and the **odd trailing byte**, which RFC 1071 pads on the right, making it the *high* byte of the last word. The complement step is why verification sums to zero: `sum + ~sum == 0xFFFF ≡ -0` in one's complement. Production stacks vectorize this (sum 32/64 bits at a time, fold once at the end), which works precisely because one's complement addition is associative and endian-agnostic.
