## challenge: Majority Element
tags: array, hash-table, counting, arrays-hashing
track: faang
difficulty: easy

Given an array `nums` of size `n`, return the element that appears more than `⌊n/2⌋` times. You may assume such a majority element always exists in the array.

Constraints: `1 <= nums.length <= 5*10^4`, `-10^9 <= nums[i] <= 10^9`, and a majority element is guaranteed.

Example: `nums = [3,2,3]` → `3`. Example: `nums = [2,2,1,1,1,2,2]` → `2` (it occupies four of seven slots).

hint: Because the majority element occupies more than half the array, it outnumbers everything else combined.
hint: Keep a running candidate and a vote counter; a value equal to the candidate is a vote for it, anything else is a vote against.
hint: When the counter hits zero, adopt the current element as the new candidate — the true majority survives this cancellation.

```cpp
// starter
#include <vector>
int majorityElement(std::vector<int>& nums);
```

```cpp
int majorityElement(std::vector<int>& nums) {
    int candidate = 0, count = 0;
    for (int x : nums) {
        if (count == 0) candidate = x;
        count += (x == candidate) ? 1 : -1;
    }
    return candidate;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{3,2,3};             if (majorityElement(n) != 3)  { std::puts("case1"); return 1; } }
    { vector<int> n{2,2,1,1,1,2,2};     if (majorityElement(n) != 2)  { std::puts("case2"); return 1; } }
    { vector<int> n{1};                 if (majorityElement(n) != 1)  { std::puts("case3"); return 1; } }
    { vector<int> n{6,6,6,7,7};         if (majorityElement(n) != 6)  { std::puts("case4"); return 1; } }
    { vector<int> n{-1,-1,-1,2,2};      if (majorityElement(n) != -1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The Boyer-Moore voting algorithm maintains a candidate and a counter. Matching votes raise the counter, differing votes lower it, and a zero counter resets the candidate. Since the majority element appears more than half the time, its net votes cannot be exhausted, so it is the final candidate. O(n) time, O(1) space.
