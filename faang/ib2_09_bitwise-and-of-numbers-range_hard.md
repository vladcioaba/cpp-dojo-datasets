## challenge: Bitwise AND of Numbers Range
tags: bit-tricks, math
track: faang
difficulty: hard

Given two integers `left` and `right` that represent the inclusive range `[left, right]`, return the bitwise AND of all numbers in this range. Because a single differing bit anywhere in the range forces that bit to 0 in the result, the answer is the common binary prefix that `left` and `right` share.

Constraints: `0 <= left <= right <= 2^31 - 1`.

Example: `left = 5, right = 7` → `4` (`5 & 6 & 7 == 4`). Example: `left = 0, right = 0` → `0`. Example: `left = 1, right = 2147483647` → `0`.

hint: Any bit that changes anywhere within the range becomes 0 in the AND, so only bits constant across `[left, right]` survive.
hint: The surviving bits are exactly the common most-significant prefix of `left` and `right`.
hint: Right-shift both numbers until they are equal, counting the shifts, then shift the common value back left by that count.

```cpp
// starter
int rangeBitwiseAnd(int left, int right);
```

```cpp
int rangeBitwiseAnd(int left, int right) {
    int shift = 0;
    while (left < right) {
        left >>= 1;
        right >>= 1;
        ++shift;
    }
    return left << shift;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (rangeBitwiseAnd(5, 7) != 4)                   { std::puts("case1"); return 1; }
    if (rangeBitwiseAnd(0, 0) != 0)                   { std::puts("case2"); return 1; }
    if (rangeBitwiseAnd(1, 2147483647) != 0)          { std::puts("case3"); return 1; }
    if (rangeBitwiseAnd(12, 15) != 12)                { std::puts("case4"); return 1; }
    if (rangeBitwiseAnd(20, 20) != 20)                { std::puts("case5"); return 1; }
    if (rangeBitwiseAnd(26, 30) != 24)                { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** For any bit position, if the range spans two numbers that differ there, the AND across the whole range zeros that bit. Thus only the leading bits that `left` and `right` hold identically can remain set — their common high-order prefix. Repeatedly shifting both values right until they coincide strips off the differing low bits; the number of shifts records how many low bits were dropped. Shifting the now-common value back left restores that prefix with zeros beneath it, which is the answer. O(log(max)) time, O(1) space.
