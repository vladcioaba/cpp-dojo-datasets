## challenge: Count Complete Subarrays in an Array
tags: array, hash-table, sliding-window
track: faang
difficulty: medium

Given an array of positive integers `nums`, a contiguous subarray is called *complete* if the number of distinct values it contains equals the number of distinct values in the whole array. Return the number of complete subarrays.

Constraints: `1 <= nums.length <= 1000`, `1 <= nums[i] <= 2000`.

Example: `nums = [1,3,1,2,2]` → `4` (the complete subarrays are `[1,3,1,2]`, `[1,3,1,2,2]`, `[3,1,2]`, `[3,1,2,2]`). Example: `nums = [5,5,5,5]` → `10` (the array has one distinct value, so every subarray is complete).

hint: Let `D` be the total number of distinct values. A subarray is complete exactly when it holds all `D` of them; it can never hold more.

hint: Counting "exactly `D` distinct" equals (total number of subarrays) minus (subarrays with at most `D - 1` distinct values).

hint: For "at most `m` distinct", slide a window that keeps its distinct count within `m` and add `right - left + 1` for each right edge.

```cpp
// starter
#include <vector>
int countCompleteSubarrays(std::vector<int>& nums);
```

```cpp
static long long atMostKDistinct(std::vector<int>& nums, int k) {
    std::unordered_map<int, int> cnt;
    int left = 0, distinct = 0;
    long long count = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        if (cnt[nums[right]]++ == 0) ++distinct;
        while (distinct > k) {
            if (--cnt[nums[left]] == 0) --distinct;
            ++left;
        }
        count += right - left + 1;
    }
    return count;
}
int countCompleteSubarrays(std::vector<int>& nums) {
    std::unordered_set<int> uniq(nums.begin(), nums.end());
    int D = uniq.size();
    long long n = nums.size();
    long long total = n * (n + 1) / 2;
    return (int)(total - atMostKDistinct(nums, D - 1));
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <unordered_set>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,3,1,2,2}; if (countCompleteSubarrays(n)!=4) { std::puts("case1"); return 1; } }
    { vector<int> n{5,5,5,5}; if (countCompleteSubarrays(n)!=10) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3,4}; if (countCompleteSubarrays(n)!=1) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,1,2,3}; if (countCompleteSubarrays(n)!=3) { std::puts("case4"); return 1; } }
    { vector<int> n{9}; if (countCompleteSubarrays(n)!=1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let `D` be the number of distinct values in the entire array. A subarray is complete iff it contains all `D` distinct values, and since it can never exceed `D`, "complete" means "exactly `D` distinct." Counting exactly-`D` directly is fiddly, so use `exactly(D) = total subarrays - atMost(D - 1)`, where the total number of subarrays is `n(n+1)/2`. The `atMost(m)` helper slides a window that keeps at most `m` distinct values, contributing `right - left + 1` per right edge. This runs in O(n) time with a hash map for the distinct counts.
