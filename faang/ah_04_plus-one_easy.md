## challenge: Plus One
tags: array, math, arrays-hashing
track: faang
difficulty: easy

You are given a non-empty array `digits` representing a non-negative integer, one decimal digit per element with the most significant digit first and no leading zeros (except the number `0` itself). Add one to the integer and return the resulting digits array.

Constraints: `1 <= digits.length <= 100`, `0 <= digits[i] <= 9`, the input has no leading zeros.

Example: `digits = [1,2,3]` → `[1,2,4]`. Example: `digits = [9,9]` → `[1,0,0]` (the carry ripples all the way and grows the array).

hint: Adding one only affects trailing `9`s, which turn into `0`s and pass a carry leftward.
hint: Walk from the least significant digit; if it is below `9` you can increment and stop immediately.
hint: If every digit was a `9` the loop finishes without returning — prepend a leading `1` to the all-zeros array.

```cpp
// starter
#include <vector>
std::vector<int> plusOne(std::vector<int>& digits);
```

```cpp
std::vector<int> plusOne(std::vector<int>& digits) {
    for (int i = (int)digits.size() - 1; i >= 0; --i) {
        if (digits[i] < 9) {
            ++digits[i];
            return digits;
        }
        digits[i] = 0;
    }
    digits.insert(digits.begin(), 1);
    return digits;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> d{1,2,3};   auto r = plusOne(d); vector<int> w{1,2,4};
      if (r != w) { std::puts("case1"); return 1; } }
    { vector<int> d{4,3,2,1}; auto r = plusOne(d); vector<int> w{4,3,2,2};
      if (r != w) { std::puts("case2"); return 1; } }
    { vector<int> d{9};       auto r = plusOne(d); vector<int> w{1,0};
      if (r != w) { std::puts("case3"); return 1; } }
    { vector<int> d{9,9};     auto r = plusOne(d); vector<int> w{1,0,0};
      if (r != w) { std::puts("case4"); return 1; } }
    { vector<int> d{1,9,9};   auto r = plusOne(d); vector<int> w{2,0,0};
      if (r != w) { std::puts("case5"); return 1; } }
    { vector<int> d{0};       auto r = plusOne(d); vector<int> w{1};
      if (r != w) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Scan from the last digit: any digit below nine can be incremented and returned right away. A nine becomes zero and the carry continues left. If the loop exhausts every digit, the number was all nines, so we insert a leading one in front of the now all-zero array. O(n) time, O(1) extra space aside from the possible resize.
