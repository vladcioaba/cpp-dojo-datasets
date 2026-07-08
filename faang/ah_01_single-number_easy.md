## challenge: Single Number
tags: array, hash-table, bit-manipulation, arrays-hashing
track: faang
difficulty: easy

You are given a non-empty array `nums` in which every value appears exactly twice, except for one value that appears only once. Return that lone value. Aim for linear time and constant extra space.

Constraints: `1 <= nums.length <= 3*10^4`, `nums.length` is odd, `-3*10^4 <= nums[i] <= 3*10^4`, and exactly one element is unpaired.

Example: `nums = [2,2,1]` → `1`. Example: `nums = [4,1,2,1,2]` → `4` (every other value cancels in pairs).

hint: A hash map counting occurrences works but costs O(n) space; there is a way to do it with no extra memory at all.
hint: XOR has two magic properties: `x ^ x == 0` and `x ^ 0 == x`, and it is commutative.
hint: Fold the whole array together with `^`; every paired value cancels to zero and only the unique value survives.

```cpp
// starter
#include <vector>
int singleNumber(std::vector<int>& nums);
```

```cpp
int singleNumber(std::vector<int>& nums) {
    int acc = 0;
    for (int x : nums) acc ^= x;
    return acc;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,2,1};       if (singleNumber(n) != 1)  { std::puts("case1"); return 1; } }
    { vector<int> n{4,1,2,1,2};   if (singleNumber(n) != 4)  { std::puts("case2"); return 1; } }
    { vector<int> n{1};           if (singleNumber(n) != 1)  { std::puts("case3"); return 1; } }
    { vector<int> n{-1,-1,-3};    if (singleNumber(n) != -3) { std::puts("case4"); return 1; } }
    { vector<int> n{0,0,7,5,5};   if (singleNumber(n) != 7)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** XOR is its own inverse, so accumulating every element with `^` makes each matched pair vanish (`x ^ x == 0`) and leaves the single unpaired value behind. One pass, O(n) time and O(1) space, with no hashing or sorting required.
