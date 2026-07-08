## challenge: Find the Duplicate Number
tags: array, two-pointers, binary-search, arrays-hashing
track: faang
difficulty: medium

You are given an array `nums` of `n + 1` integers where every value lies in the range `[1, n]`. By the pigeonhole principle at least one value is repeated; in fact exactly one value is repeated, though it may appear more than twice. Return that repeated value without modifying the array and using only O(1) extra space.

Constraints: `1 <= n <= 10^5`, `nums.length == n + 1`, `1 <= nums[i] <= n`, exactly one value repeats.

Example: `nums = [1,3,4,2,2]` → `2`. Example: `nums = [3,1,3,4,2]` → `3`.

hint: Because values are in `[1, n]` and indices in `[0, n]`, treating each value as a "next index" turns the array into a linked structure that must contain a cycle.
hint: The duplicated value is exactly the entry point of that cycle, so Floyd's tortoise-and-hare cycle detection applies.
hint: Advance a slow pointer one step and a fast pointer two steps until they meet, then restart one pointer at the head and step both by one until they meet again — that meeting point is the answer.

```cpp
// starter
#include <vector>
int findDuplicate(std::vector<int>& nums);
```

```cpp
int findDuplicate(std::vector<int>& nums) {
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,3,4,2,2};       if (findDuplicate(n) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{3,1,3,4,2};       if (findDuplicate(n) != 3) { std::puts("case2"); return 1; } }
    { vector<int> n{1,1};             if (findDuplicate(n) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{2,2,2,2,2};       if (findDuplicate(n) != 2) { std::puts("case4"); return 1; } }
    { vector<int> n{4,3,1,4,2};       if (findDuplicate(n) != 4) { std::puts("case5"); return 1; } }
    { vector<int> n{2,5,9,6,9,3,8,9,7,1}; if (findDuplicate(n) != 9) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Reading `nums[i]` as the successor of node `i` builds a functional graph where the repeated value is the entrance of a cycle (multiple indices point to it). Floyd's algorithm first finds a meeting point inside the cycle with slow/fast pointers, then walks one pointer from the start in lockstep to locate the cycle entry, which is the duplicate. O(n) time, O(1) space, and the array is never mutated.
