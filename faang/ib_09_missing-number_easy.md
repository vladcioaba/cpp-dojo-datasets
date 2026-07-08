## challenge: Missing Number
tags: bit-tricks, math, array
track: faang
difficulty: easy

Given an array `nums` containing `n` distinct numbers drawn from the range `[0, n]`, exactly one number in that range is missing from the array. Return the missing number using constant extra space.

Constraints: `n == nums.length`, `1 <= n <= 10^4`, `0 <= nums[i] <= n`, all values in `nums` are distinct.

Example: `nums = [3,0,1]` → `2`. Example: `nums = [0,1]` → `2`. Example: `nums = [9,6,4,2,3,5,7,0,1]` → `8`.

hint: The full set `0..n` and the array differ by exactly one value; think about what cancels.
hint: XOR every index `0..n` together with every array value. Each present number appears once as an index and once as a value, so it cancels.
hint: Seed the accumulator with `n`, then fold in `i ^ nums[i]` for each position; the survivor is the missing number.

```cpp
// starter
#include <vector>
int missingNumber(std::vector<int>& nums);
```

```cpp
int missingNumber(std::vector<int>& nums) {
    int n = (int)nums.size();
    int result = n;
    for (int i = 0; i < n; ++i)
        result ^= i ^ nums[i];
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
    { vector<int> n{3,0,1};                 if (missingNumber(n) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{0,1};                   if (missingNumber(n) != 2) { std::puts("case2"); return 1; } }
    { vector<int> n{9,6,4,2,3,5,7,0,1};     if (missingNumber(n) != 8) { std::puts("case3"); return 1; } }
    { vector<int> n{0};                     if (missingNumber(n) != 1) { std::puts("case4"); return 1; } }
    { vector<int> n{1};                     if (missingNumber(n) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Consider the multiset of all indices `0..n` combined with all array values. Every number that is present contributes itself twice — once as an index it occupies and once as the value stored — so it XORs away to 0. The one number never stored as a value contributes only once (as an index) and survives. Starting the accumulator at `n` covers the extra index `n` that the loop over positions `0..n-1` cannot reach. O(n) time, O(1) space, and no risk of overflow that a summation approach might invite.
