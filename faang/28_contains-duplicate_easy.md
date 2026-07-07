## challenge: Contains Duplicate
tags: array, hash-table, arrays-hashing
track: faang
difficulty: easy

Given an integer array `nums`, return `true` if any value appears at least twice in the array, and return `false` if every element is distinct.

Constraints: `0 <= nums.length <= 10^5`, `-10^9 <= nums[i] <= 10^9`.

Example: `nums = [1,2,3,1]` → `true` (the value `1` appears twice). Example: `nums = [1,2,3,4]` → `false`. Example: `nums = []` → `false`.

hint: You only need to know whether a value has been seen before, not how many times or where.
hint: A hash set gives O(1) average insertion and membership checks.
hint: Walk the array once; if inserting an element reports it was already present, return `true`.

```cpp
// starter
#include <vector>
bool containsDuplicate(std::vector<int>& nums);
```

```cpp
bool containsDuplicate(std::vector<int>& nums) {
    std::unordered_set<int> seen;
    seen.reserve(nums.size() * 2);
    for (int x : nums) {
        if (!seen.insert(x).second) return true;
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_set>
using std::vector;
//__USER__
int main() {
    { vector<int> n{};              if ( containsDuplicate(n)) { std::puts("case1"); return 1; } }
    { vector<int> n{7};             if ( containsDuplicate(n)) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3,4};       if ( containsDuplicate(n)) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,3,1};       if (!containsDuplicate(n)) { std::puts("case4"); return 1; } }
    { vector<int> n{-1,0,1,-1};     if (!containsDuplicate(n)) { std::puts("case5"); return 1; } }
    { vector<int> n{5,5};           if (!containsDuplicate(n)) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Insert each value into a hash set as you scan; `unordered_set::insert` returns whether the element was newly added, so a failed insertion means a duplicate. This runs in O(n) average time and O(n) space, beating the O(n^2) all-pairs comparison and the O(n log n) sort-and-scan alternative.
