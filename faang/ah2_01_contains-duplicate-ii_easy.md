## challenge: Contains Duplicate II
tags: array, hash-table, sliding-window, arrays-hashing
track: faang
difficulty: easy

Given an integer array `nums` and an integer `k`, return `true` if there are two distinct indices `i` and `j` such that `nums[i] == nums[j]` and `abs(i - j) <= k`. Otherwise return `false`.

Constraints: `1 <= nums.length <= 10^5`, `-10^9 <= nums[i] <= 10^9`, `0 <= k <= 10^5`.

Example: `nums = [1,2,3,1], k = 3` → `true` (the two 1's are 3 indices apart). Example: `nums = [1,0,1,1], k = 1` → `true`. Example: `nums = [1,2,3,1,2,3], k = 2` → `false` (equal values are 3 indices apart).

hint: A brute-force check of every pair within a window of size `k` is O(n*k); for each value you only ever care about the most recent position you saw it at.
hint: Keep a hash map from value to the last index where it appeared, and update it as you scan left to right.
hint: When you meet a value already in the map, compare the current index with the stored one; if the gap is at most `k` return true, otherwise overwrite the entry with the newer index.

```cpp
// starter
#include <vector>
bool containsNearbyDuplicate(std::vector<int>& nums, int k);
```

```cpp
bool containsNearbyDuplicate(std::vector<int>& nums, int k) {
    std::unordered_map<int, int> last;
    for (int i = 0; i < (int)nums.size(); ++i) {
        auto it = last.find(nums[i]);
        if (it != last.end() && i - it->second <= k) return true;
        last[nums[i]] = i;
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,3,1}; if (containsNearbyDuplicate(n, 3) != true) { std::puts("case1"); return 1; } }
    { vector<int> n{1,0,1,1}; if (containsNearbyDuplicate(n, 1) != true) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3,1,2,3}; if (containsNearbyDuplicate(n, 2) != false) { std::puts("case3"); return 1; } }
    { vector<int> n{99}; if (containsNearbyDuplicate(n, 0) != false) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3,1,2,3}; if (containsNearbyDuplicate(n, 3) != true) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Slide a "last seen" hash map across the array. For each element look up its previous index; if it exists and the distance to the current index is within `k`, a qualifying pair exists. Otherwise store the current index, always keeping only the freshest occurrence because a nearer duplicate can only help. One pass, average O(1) per operation, O(n) time and O(min(n,k)) space.
