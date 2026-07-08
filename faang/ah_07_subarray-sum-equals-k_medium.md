## challenge: Subarray Sum Equals K
tags: array, hash-table, prefix-sum, arrays-hashing
track: faang
difficulty: medium

Given an integer array `nums` and an integer `k`, return the number of contiguous, non-empty subarrays whose elements sum to exactly `k`. The array may contain negative numbers, so a sliding window will not work.

Constraints: `1 <= nums.length <= 2*10^4`, `-1000 <= nums[i] <= 1000`, `-10^7 <= k <= 10^7`.

Example: `nums = [1,1,1], k = 2` → `2` (the two overlapping `[1,1]` windows). Example: `nums = [1,2,3], k = 3` → `2` (`[1,2]` and `[3]`).

hint: A subarray sum equals `prefix[j] - prefix[i]`; you want that difference to be `k`.
hint: Rearranged, you need a previous prefix sum equal to `current - k`, so count how many times each prefix sum has occurred in a hash map.
hint: Seed the map with prefix sum `0` counted once (the empty prefix) so subarrays starting at index 0 are counted.

```cpp
// starter
#include <vector>
int subarraySum(std::vector<int>& nums, int k);
```

```cpp
int subarraySum(std::vector<int>& nums, int k) {
    std::unordered_map<long long, int> seen;
    seen[0] = 1;
    long long prefix = 0;
    int count = 0;
    for (int x : nums) {
        prefix += x;
        auto it = seen.find(prefix - k);
        if (it != seen.end()) count += it->second;
        ++seen[prefix];
    }
    return count;
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
    { vector<int> n{1,1,1};                 if (subarraySum(n, 2) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,3};                 if (subarraySum(n, 3) != 2) { std::puts("case2"); return 1; } }
    { vector<int> n{-1,-1,1};               if (subarraySum(n, 0) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{0,0,0};                 if (subarraySum(n, 0) != 6) { std::puts("case4"); return 1; } }
    { vector<int> n{3,4,7,2,-3,1,4,2};      if (subarraySum(n, 7) != 4) { std::puts("case5"); return 1; } }
    { vector<int> n{1};                     if (subarraySum(n, 0) != 0) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let `prefix` be the running sum up to the current index. A subarray ending here sums to `k` exactly when some earlier prefix equalled `prefix - k`, so we keep a hash map from prefix-sum value to how many times it has appeared and add that count. Seeding `prefix 0` with count one handles subarrays that begin at index zero. One pass, O(n) time and O(n) space, and it correctly handles negatives.
