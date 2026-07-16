## challenge: Message length: table, not switch
tags: branch-prediction, dispatch, hot-path
track: hft
difficulty: medium

An ITCH-style feed parser must map a message type byte to its wire length before it can advance to the next message — on every single packet. Implement `uint8_t msgLength(uint8_t type)` for this dictionary: `'A'` (add) → 36, `'E'` (execute) → 31, `'X'` (cancel) → 23, `'D'` (delete) → 19, `'U'` (replace) → 34, `'P'` (trade) → 44; any other byte → 0. No `switch`, no `if` chain: build a 256-entry `constexpr` table at compile time and make the function body a single array load.

Constraints: the table must be `constexpr` (built at compile time, stored in read-only data); `msgLength` contains exactly one indexed load and no control flow; input is any byte 0–255.

Example: `msgLength('A') == 36`, `msgLength('P') == 44`, `msgLength('Z') == 0`, `msgLength(0) == 0`.

hint: An immediately-invoked `constexpr` lambda builds the table cleanly: `constexpr auto kLen = []{ std::array<uint8_t,256> t{}; ...; return t; }();` — value-initialization zeroes all 256 slots first.
hint: Assign only the six known types (`t['A'] = 36;` etc.); every unknown byte stays at the zero the initialization gave it.
hint: `uint8_t` indexes cover 0–255 exactly, so `kLen[type]` needs no bounds check — the domain of the input *is* the domain of the table.

```cpp
// starter
#include <cstdint>
#include <array>
uint8_t msgLength(uint8_t type);
```

```cpp
constexpr auto kLen = [] {
    std::array<uint8_t, 256> t{};   // all zeros: unknown types map to 0
    t['A'] = 36;
    t['E'] = 31;
    t['X'] = 23;
    t['D'] = 19;
    t['U'] = 34;
    t['P'] = 44;
    return t;
}();

uint8_t msgLength(uint8_t type) {
    return kLen[type];
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <array>
//__USER__
int main() {
    struct { uint8_t type; uint8_t want; } cases[] = {
        {'A', 36}, {'E', 31}, {'X', 23}, {'D', 19}, {'U', 34}, {'P', 44},
        {'Z', 0}, {'a', 0}, {'p', 0}, {0, 0}, {255, 0}, {' ', 0},
    };
    for (auto& c : cases) {
        uint8_t got = msgLength(c.type);
        if (got != c.want) {
            std::printf("msgLength(%d)=%d want %d\n", (int)c.type, (int)got, (int)c.want);
            return 1;
        }
    }
    unsigned known = 0;
    for (int t = 0; t < 256; ++t) known += (msgLength((uint8_t)t) != 0) ? 1u : 0u;
    if (known != 6) { std::printf("%u nonzero entries, want exactly 6\n", known); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A `switch` over sparse byte values compiles to either a compare-and-branch chain or a jump table — and a jump table is an *indirect branch*, which the predictor must guess by target. A market-data stream interleaves adds, executes, and cancels unpredictably, so that indirect branch mispredicts constantly, costing ~15–20 cycles per message before you've parsed a single field. The dense lookup table replaces all control flow with one data load: 256 bytes is four cache lines, hot forever in L1, so the lookup is ~4–5 cycles with perfectly flat latency — no best case, no worst case. Building it with an immediately-invoked `constexpr` lambda means zero runtime initialization: the table is baked into `.rodata` at compile time, and mistakes like a wrong length are still checkable with `static_assert`. Sizing the table to exactly 256 entries and indexing with `uint8_t` eliminates the bounds check by construction — the type system proves the index is in range. This pattern (byte → attribute via flat table) is everywhere in feed handlers: message lengths, field offsets, per-type dispatch indices; when handlers are functions, the same table stores function pointers, trading the switch for one indirect call that at least the branch-target buffer can learn per-site.
