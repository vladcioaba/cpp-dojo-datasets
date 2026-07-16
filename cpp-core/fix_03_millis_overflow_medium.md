## challenge: fix: uptime goes backwards after 49 days
tags: code-review, debugging, integer-arithmetic
track: core
difficulty: medium

This code review found a bug: the reported uptime in milliseconds is correct for weeks, then suddenly jumps back near zero once the server has been up for about 49 days. Find and fix it — keep the function signature.

hint: The return type is 64-bit, but where does the arithmetic actually happen?
hint: This is an integer overflow: the multiplication wraps before the widening conversion.
hint: uint32_t * int is computed in 32 bits and wraps at 2^32 (~49.7 days of milliseconds); only afterwards is the wrapped value converted to uint64_t.

```cpp
// starter
std::uint64_t uptimeMillis(std::uint32_t uptimeSeconds) {
    return uptimeSeconds * 1000;
}
```

```cpp
std::uint64_t uptimeMillis(std::uint32_t uptimeSeconds) {
    return static_cast<std::uint64_t>(uptimeSeconds) * 1000;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(uptimeMillis(0) == 0);
    assert(uptimeMillis(2) == 2000);
    assert(uptimeMillis(4294967) == 4294967000ULL);      // just under the 2^32 ms line
    assert(uptimeMillis(5000000) == 5000000000ULL);      // ~58 days: wraps in the buggy version
    std::puts("PASS");
}
```

**Editorial:** `uptimeSeconds * 1000` is evaluated in `std::uint32_t` (the `int` literal converts to unsigned of the same rank), so the product is reduced modulo 2^32 — 5,000,000 seconds becomes 705,032,704 ms instead of 5,000,000,000 ms. The 64-bit return type widens the value only *after* it has already wrapped. Widen an operand first: `static_cast<std::uint64_t>(uptimeSeconds) * 1000`. Reviewers should be suspicious whenever a narrow multiplication feeds a wide result type — the destination type never rescues the arithmetic.
