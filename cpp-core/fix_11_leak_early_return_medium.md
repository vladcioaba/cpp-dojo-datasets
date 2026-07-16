## challenge: fix: the packet validator that leaks
tags: code-review, debugging, raii
track: core
difficulty: medium

This code review found a bug: a long-running service that rejects many malformed packets slowly exhausts memory — the live-buffer counter climbs and never comes back down. Find and fix it — keep the function signature.

hint: Follow every return path and ask which ones run the delete.
hint: This is a resource leak on an early return.
hint: The validation early-return exits before `delete buf`, so every rejected packet leaks one Buffer; RAII (std::unique_ptr) makes all exits safe.

```cpp
// starter
struct Buffer {
    inline static int alive = 0;
    std::array<int, 64> data{};
    Buffer() { ++alive; }
    ~Buffer() { --alive; }
};

bool processPacket(const std::vector<int>& packet) {
    Buffer* buf = new Buffer();
    if (packet.empty() || packet.size() > buf->data.size()) {
        return false;   // reject malformed packet
    }
    for (std::size_t i = 0; i < packet.size(); ++i) {
        buf->data[i] = packet[i];
    }
    bool ok = buf->data[0] == 42;
    delete buf;
    return ok;
}
```

```cpp
struct Buffer {
    inline static int alive = 0;
    std::array<int, 64> data{};
    Buffer() { ++alive; }
    ~Buffer() { --alive; }
};

bool processPacket(const std::vector<int>& packet) {
    auto buf = std::make_unique<Buffer>();
    if (packet.empty() || packet.size() > buf->data.size()) {
        return false;   // reject malformed packet
    }
    for (std::size_t i = 0; i < packet.size(); ++i) {
        buf->data[i] = packet[i];
    }
    return buf->data[0] == 42;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(processPacket({42, 7, 9}) == true);
    assert(processPacket({7}) == false);      // valid size, wrong magic: cleaned up in both versions
    assert(Buffer::alive == 0);
    assert(processPacket({}) == false);       // malformed: early return
    assert(Buffer::alive == 0);               // buggy version leaked one Buffer here
    std::puts("PASS");
}
```

**Editorial:** The happy path pairs `new Buffer()` with `delete buf`, but the malformed-packet branch returns *before* the delete, so every rejected packet leaks a `Buffer` — invisible in tests that only send valid traffic, fatal in a service bombarded with junk. Deleting before the early return would patch it, but the robust fix is RAII: hold the buffer in a `std::unique_ptr` (or just declare it on the stack here) so *every* exit path — including future returns and exceptions — releases it. Reviewers should treat any raw `new` in a function with multiple returns as guilty until proven paired on all paths.
