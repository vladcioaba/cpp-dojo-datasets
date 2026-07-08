## challenge: Palindrome Number
tags: math
track: faang
difficulty: easy

Given an integer `x`, return `true` if `x` reads the same backward as forward, and `false` otherwise. Solve it without converting the integer to a string. Any negative number is not a palindrome because of the leading minus sign, and a positive number ending in 0 cannot be a palindrome (its reverse would have a leading zero).

Constraints: `-2^31 <= x <= 2^31 - 1`.

Example: `x = 121` → `true`. Example: `x = -121` → `false` (reads `121-`). Example: `x = 10` → `false`.

hint: Reversing the whole number could overflow; reverse only the second half and compare it to the first half.
hint: Negatives are never palindromes, and any positive multiple of 10 (except 0) cannot be either.
hint: Pop digits from `x` into a reversed accumulator until that accumulator is at least `x`; then `x == rev` (even length) or `x == rev / 10` (odd length) decides it.

```cpp
// starter
bool isPalindrome(int x);
```

```cpp
bool isPalindrome(int x) {
    if (x < 0 || (x % 10 == 0 && x != 0)) return false;
    long long rev = 0;
    while (x > rev) {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    return x == rev || x == rev / 10;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (isPalindrome(121) != true)    { std::puts("case1"); return 1; }
    if (isPalindrome(-121) != false)  { std::puts("case2"); return 1; }
    if (isPalindrome(10) != false)    { std::puts("case3"); return 1; }
    if (isPalindrome(0) != true)      { std::puts("case4"); return 1; }
    if (isPalindrome(12321) != true)  { std::puts("case5"); return 1; }
    if (isPalindrome(1221) != true)   { std::puts("case6"); return 1; }
    if (isPalindrome(100) != false)   { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Rejecting negatives and non-zero multiples of 10 up front removes the tricky sign and trailing-zero cases. Then peel digits off the back of `x` into `rev`, stopping once `rev >= x`; at that point `rev` holds the reversed lower half and `x` the upper half. For an even number of digits the two halves match exactly (`x == rev`); for an odd count the middle digit sits in `rev`, so `x == rev / 10` drops it before comparing. Only half the digits are processed, giving O(log x) time and O(1) space with no overflow.
