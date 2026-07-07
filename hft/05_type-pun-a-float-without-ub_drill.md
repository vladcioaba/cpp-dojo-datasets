## challenge: Type-pun a float without UB
tags: undefined-behavior, aliasing, bit-cast
track: hft

Reinterpreting a `float` as its bits via `*(uint32_t*)&f` is a strict-aliasing violation — UB the optimizer can miscompile. The correct, zero-cost way is `std::memcpy` (or C++20 `std::bit_cast`). Implement `uint32_t float_bits(float f)` returning the IEEE-754 bit pattern with no aliasing violation.

hint: Reading a float's storage through an unrelated pointer type breaks strict aliasing; the fix is to copy the bytes, not reinterpret them.
hint: `std::memcpy` into a same-sized integer is the blessed, zero-cost idiom (C++20 adds `std::bit_cast`).

```cpp
uint32_t float_bits(float f) {
    uint32_t bits;
    std::memcpy(&bits, &f, sizeof bits);
    return bits;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstring>
//__USER__
int main() {
    if (float_bits(1.0f)  != 0x3f800000u) { std::puts("1.0f wrong"); return 1; }
    if (float_bits(0.0f)  != 0x00000000u) { std::puts("0.0f wrong"); return 1; }
    if (float_bits(-2.0f) != 0xc0000000u) { std::puts("-2.0f wrong"); return 1; }
    if (float_bits(2.0f)  != 0x40000000u) { std::puts("2.0f wrong"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** `std::memcpy` copies the object representation into a `uint32_t` without ever forming a pointer that violates strict aliasing, so the optimizer cannot miscompile it — and it lowers to a single register move. This is the standard way to type-pun; `std::bit_cast` is the constexpr C++20 equivalent. O(1).
