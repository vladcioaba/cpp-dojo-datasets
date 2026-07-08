## challenge: Number of Subsequences That Satisfy the Given Sum Condition
tags: two-pointers, array, sorting, math
track: faang
difficulty: hard

Given an array of integers `nums` and an integer `target`, count the number of non-empty subsequences such that the sum of the minimum and maximum element of the subsequence is less than or equal to `target`. Since the answer may be very large, return it modulo `10^9 + 7`.

Constraints: `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^6`, `1 <= target <= 10^6`.

Example: `nums = [3,5,6,7], target = 9` → `4`. Example: `nums = [3,3,6,8], target = 10` → `6`. Example: `nums = [2,3,3,4,6,7], target = 12` → `61`.

hint: Only the minimum and maximum of a subsequence matter, so sorting does not change any answer and lets you reason about pairs of endpoints.
hint: With two pointers on the sorted array, if `nums[lo] + nums[hi] <= target`, then `nums[lo]` can pair with any subset of the elements strictly between `lo` and `hi`.
hint: That contributes `2^(hi-lo)` subsequences; precompute the powers of two modulo `10^9 + 7` to add them in O(1).

```cpp
// starter
#include <vector>
int numSubseq(std::vector<int>& nums, int target);
```

```cpp
int numSubseq(std::vector<int>& nums, int target) {
    const int MOD = 1000000007;
    int n = (int)nums.size();
    std::sort(nums.begin(), nums.end());
    std::vector<long long> pow2(n);
    pow2[0] = 1;
    for (int i = 1; i < n; ++i) pow2[i] = pow2[i - 1] * 2 % MOD;
    long long res = 0;
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        if (nums[lo] + nums[hi] <= target) {
            res = (res + pow2[hi - lo]) % MOD;
            ++lo;
        } else {
            --hi;
        }
    }
    return (int)res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{3,5,6,7}; if (numSubseq(n, 9) != 4) { std::puts("case1"); return 1; } }
    { vector<int> n{3,3,6,8}; if (numSubseq(n, 10) != 6) { std::puts("case2"); return 1; } }
    { vector<int> n{2,3,3,4,6,7}; if (numSubseq(n, 12) != 61) { std::puts("case3"); return 1; } }
    { vector<int> n{5,2,4,1,7,6,8}; if (numSubseq(n, 16) != 127) { std::puts("case4"); return 1; } }
    { vector<int> n{1}; if (numSubseq(n, 2) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because a subsequence is judged only by its smallest and largest element, sorting is free and turns the condition into a statement about two endpoints. Sweep with `lo` and `hi`: if `nums[lo] + nums[hi] <= target`, then fixing `nums[lo]` as the minimum, every one of the `hi - lo` elements between the pointers may be independently included or not, yielding `2^(hi-lo)` valid subsequences, after which advance `lo`; otherwise the current maximum is too large, so decrement `hi`. Precomputed powers of two modulo `10^9 + 7` make each step O(1). O(n log n) time from sorting, O(n) space.
