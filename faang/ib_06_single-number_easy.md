## challenge: Single Number
tags: bit-tricks, array, hash-table
track: faang
difficulty: easy

Given a non-empty array `nums` in which every element appears exactly twice except for one element that appears once, find that single element. Your solution must run in linear time and use only constant extra space.

Constraints: `1 <= nums.length <= 3*10^4`, `nums.length` is odd, `-3*10^4 <= nums[i] <= 3*10^4`, exactly one element is unpaired.

Example: `nums = [2,2,1]` → `1`. Example: `nums = [4,1,2,1,2]` → `4`. Example: `nums = [1]` → `1`.

hint: A hash set works but costs O(n) space; the constraints hint at something cheaper.
hint: XOR has two properties that matter: `x ^ x == 0` and `x ^ 0 == x`, and it is commutative and associative.
hint: XOR every element together. All paired values cancel to 0, leaving only the lone element.

```cpp
// starter
#include <vector>
int singleNumber(std::vector<int>& nums);
```

```cpp
int singleNumber(std::vector<int>& nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result;
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
    { vector<int> n{-1,-1,-2};    if (singleNumber(n) != -2) { std::puts("case4"); return 1; } }
    { vector<int> n{7,3,3,7,11};  if (singleNumber(n) != 11) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** XOR is its own inverse: any value XORed with itself is 0, and XOR with 0 is the identity. Because XOR is commutative and associative, folding the whole array with `^` makes every duplicated pair vanish, and only the unique value remains. This gives O(n) time and O(1) space with no hashing.
