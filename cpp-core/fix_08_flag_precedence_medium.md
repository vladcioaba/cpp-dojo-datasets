## challenge: fix: permissions only work for bit zero
tags: code-review, debugging, operators
track: core
difficulty: medium

This code review found a bug: permission checks pass or fail based only on whether the user has the READ bit — asking for WRITE with WRITE granted returns false, while asking for EXEC with only READ granted returns true. Find and fix it — keep the function signature.

hint: The bitmask logic is conceptually right; the compiler just isn't grouping it the way the author thought.
hint: This is an operator-precedence bug: == binds tighter than &.
hint: `flags & required == required` parses as `flags & (required == required)`, i.e. `flags & 1` — it only ever tests the lowest bit of flags.

```cpp
// starter
// Returns true when every bit set in `required` is also set in `flags`.
bool hasPermission(unsigned flags, unsigned required) {
    return flags & required == required;
}
```

```cpp
// Returns true when every bit set in `required` is also set in `flags`.
bool hasPermission(unsigned flags, unsigned required) {
    return (flags & required) == required;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    constexpr unsigned FLAG_READ  = 1u << 0;
    constexpr unsigned FLAG_WRITE = 1u << 1;
    constexpr unsigned FLAG_EXEC  = 1u << 2;

    assert(hasPermission(FLAG_WRITE, FLAG_WRITE));                  // buggy: false
    assert(!hasPermission(FLAG_READ, FLAG_EXEC));                   // buggy: true
    assert(hasPermission(FLAG_READ | FLAG_WRITE, FLAG_WRITE));
    assert(!hasPermission(FLAG_WRITE, FLAG_WRITE | FLAG_EXEC));
    assert(hasPermission(FLAG_READ, FLAG_READ));
    std::puts("PASS");
}
```

**Editorial:** Equality binds tighter than bitwise AND, so `flags & required == required` evaluates `required == required` first — always `true`, which converts to `1` — and the whole check collapses to `flags & 1`: "does the user have the READ bit". Parenthesize the mask test: `(flags & required) == required`. This precedence quirk (relational and equality operators binding tighter than `&`, `^`, `|`) is a C legacy famous enough that compilers emit `-Wparentheses` for it; in review, any bitwise expression mixed with a comparison and no parentheses should be re-derived by hand.
