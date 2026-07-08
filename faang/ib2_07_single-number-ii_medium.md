## challenge: Single Number II
tags: bit-tricks, array
track: faang
difficulty: medium

Given an integer array `nums` where every element appears exactly three times except for one element, which appears exactly once, find and return the single element. Your algorithm must run in linear time and use only constant extra space.

Constraints: `1 <= nums.length <= 3*10^4`, `-2^31 <= nums[i] <= 2^31 - 1`, every element appears three times except one that appears once.

Example: `nums = [2,2,3,2]` → `3`. Example: `nums = [0,1,0,1,0,1,99]` → `99`. Example: `nums = [1]` → `1`.

hint: Counting each bit's occurrences modulo 3 isolates the bits of the unique number.
hint: Two accumulators, `ones` and `twos`, can act as a base-3 counter per bit: `ones` holds bits seen once, `twos` holds bits seen twice, and a bit seen three times clears from both.
hint: The update `ones = (ones ^ x) & ~twos; twos = (twos ^ x) & ~ones;` keeps this invariant; after the loop `ones` is the answer.

```cpp
// starter
#include <vector>
int singleNumber(std::vector<int>& nums);
```

```cpp
int singleNumber(std::vector<int>& nums) {
    int ones = 0, twos = 0;
    for (int x : nums) {
        ones = (ones ^ x) & ~twos;
        twos = (twos ^ x) & ~ones;
    }
    return ones;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,2,3,2};                if (singleNumber(n) != 3)   { std::puts("case1"); return 1; } }
    { vector<int> n{0,1,0,1,0,1,99};         if (singleNumber(n) != 99)  { std::puts("case2"); return 1; } }
    { vector<int> n{1};                       if (singleNumber(n) != 1)   { std::puts("case3"); return 1; } }
    { vector<int> n{-2,-2,-2,5};             if (singleNumber(n) != 5)   { std::puts("case4"); return 1; } }
    { vector<int> n{30000,500,100,30000,100,30000,100}; if (singleNumber(n) != 500) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Track each bit's count modulo 3 with two bitmasks. `ones` marks bits that have appeared once (mod 3) and `twos` marks bits that have appeared twice. Processing a value `x`, `ones = (ones ^ x) & ~twos` folds `x` into the "seen once" set unless it is already in "seen twice", and the symmetric update advances `twos`. When a bit reaches a third appearance it is cleared from both masks, so after the full pass only the bits belonging to the unique element remain in `ones`. O(n) time, O(1) space.
