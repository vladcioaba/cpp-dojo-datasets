## challenge: Single Number III
tags: bit-tricks, array
track: faang
difficulty: medium

Given an integer array `nums` in which exactly two elements appear only once and all the other elements appear exactly twice, return the two elements that appear only once. You may return the answer in any order. Your algorithm should run in linear time and use only constant extra space.

Constraints: `2 <= nums.length <= 3*10^4`, `-2^31 <= nums[i] <= 2^31 - 1`, exactly two elements appear once and the rest appear twice.

Example: `nums = [1,2,1,3,2,5]` → `[3,5]`. Example: `nums = [-1,0]` → `[-1,0]`. Example: `nums = [0,1]` → `[0,1]`.

hint: XOR of the whole array equals `a ^ b`, the XOR of the two unique numbers.
hint: Any set bit of `a ^ b` is a position where `a` and `b` differ; isolate the lowest such bit with `xr & -xr`.
hint: Partition all numbers by that bit and XOR each group separately; each group cancels its duplicates and yields one unique number.

```cpp
// starter
#include <vector>
std::vector<int> singleNumber(std::vector<int>& nums);
```

```cpp
std::vector<int> singleNumber(std::vector<int>& nums) {
    unsigned int xr = 0;
    for (int x : nums) xr ^= (unsigned int)x;
    unsigned int bit = xr & (0u - xr);
    int a = 0, b = 0;
    for (int x : nums) {
        if ((unsigned int)x & bit) a ^= x;
        else b ^= x;
    }
    return {a, b};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
static bool eq2(vector<int> r, int x, int y) {
    std::sort(r.begin(), r.end());
    if (x > y) std::swap(x, y);
    return r.size() == 2 && r[0] == x && r[1] == y;
}
int main() {
    { vector<int> n{1,2,1,3,2,5}; if (!eq2(singleNumber(n), 3, 5))   { std::puts("case1"); return 1; } }
    { vector<int> n{-1,0};         if (!eq2(singleNumber(n), -1, 0))  { std::puts("case2"); return 1; } }
    { vector<int> n{0,1};          if (!eq2(singleNumber(n), 0, 1))   { std::puts("case3"); return 1; } }
    { vector<int> n{9,9,7,-3};     if (!eq2(singleNumber(n), 7, -3))  { std::puts("case4"); return 1; } }
    { vector<int> n{1,1,2,2,3,4};  if (!eq2(singleNumber(n), 3, 4))   { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** XORing every element cancels the paired values and leaves `xr = a ^ b`, where `a` and `b` are the two singletons. Since `a != b`, `xr` has at least one set bit; `xr & -xr` isolates its lowest set bit, a position where `a` and `b` disagree. Splitting the array into numbers with that bit set versus clear places `a` in one group and `b` in the other, while every duplicated pair stays together. XORing each group independently recovers the two answers. Unsigned arithmetic makes the `-xr` well defined. O(n) time, O(1) space.
