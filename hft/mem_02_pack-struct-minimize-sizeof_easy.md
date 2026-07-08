## challenge: Pack a struct to minimize sizeof
tags: struct-packing, sizeof, alignment
track: hft
difficulty: easy

A struct's size depends on member *order*: the compiler inserts padding so each member sits on its natural alignment, and rounds the whole struct up to its largest member's alignment. Declared carelessly, this struct wastes bytes to padding. Reorder its members — largest alignment first — so `sizeof(Packed)` is as small as possible. Keep all five members with the same names and types: `double d`, `int i`, `short s`, `char a`, `char b`.

Constraints: only reordering is allowed (no bitfields, no `#pragma pack`, no changing types). The minimum achievable size is 16 bytes on a typical 64-bit ABI.

Example: a naive order like `char, double, char, int, short` balloons to 32 bytes; sorting members from widest to narrowest packs the same data into 16.

hint: Total payload here is `8 + 4 + 2 + 1 + 1 = 16` bytes — the goal is a layout with zero interior padding.
hint: Place members in non-increasing alignment order: the `double` (8) first, then `int` (4), then `short` (2), then the two `char`s.
hint: The struct's own alignment equals its widest member (8 for the `double`); once the payload already totals a multiple of 8, no tail padding is added.

```cpp
// starter
struct Packed {
    // Reorder these five members to minimize sizeof(Packed).
    char a;
    double d;
    char b;
    int i;
    short s;
};
```

```cpp
struct Packed {
    double d;   // 8 bytes, widest alignment -> goes first
    int i;      // 4
    short s;    // 2
    char a;     // 1
    char b;     // 1
};              // 8+4+2+1+1 = 16, no padding
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
//__USER__
int main() {
    static_assert(sizeof(Packed) == 16, "should pack into 16 bytes with no wasted padding");
    static_assert(alignof(Packed) == 8, "the double forces 8-byte alignment");
    Packed p{};
    p.d = 3.5; p.i = 7; p.s = 9; p.a = 'x'; p.b = 'y';
    if (p.d != 3.5 || p.i != 7 || p.s != 9 || p.a != 'x' || p.b != 'y') {
        std::puts("member read/write broken"); return 1;
    }
    std::puts("PASS");
}
```

**Editorial:** Each member must land on an offset that is a multiple of its alignment, so a narrow member ahead of a wide one forces the compiler to insert padding to realign. Ordering members from widest to narrowest lets every field abut the previous one with no gaps, and because the running offset stays a multiple of the next member's alignment there is no interior padding. Here the payload sums to 16, already a multiple of the struct's 8-byte alignment, so no tail padding is needed either. The lesson for hot-path data: sort fields by alignment to shrink structs, fit more per cache line, and cut memory traffic.
