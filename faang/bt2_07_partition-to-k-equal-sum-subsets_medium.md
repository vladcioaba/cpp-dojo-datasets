## challenge: Partition to K Equal Sum Subsets
tags: backtracking, bitmask, dynamic-programming
track: faang
difficulty: medium

Given an integer array `nums` and an integer `k`, return `true` if it is possible to divide the array into `k` non-empty subsets whose sums are all equal. Every element must belong to exactly one subset.

Constraints: `1 <= k <= nums.length <= 16`, `1 <= nums[i] <= 10^4`.

Example: `nums = [4,3,2,3,5,2,1], k = 4` -> `true` (subsets `(5)`, `(1,4)`, `(2,3)`, `(2,3)`, each summing to 5). Example: `nums = [1,2,3,4], k = 3` -> `false`. Example: `nums = [1,2,3,4], k = 1` -> `true`.

hint: Every subset must sum to `total / k`, so bail out at once if `total` is not divisible by `k` or any element exceeds that target.
hint: Fill one subset at a time; when the current subset reaches the target, recurse to begin the next one.
hint: Sort descending, and when the current subset is empty give up early if the largest unused element cannot start a valid subset.

```cpp
// starter
#include <vector>
bool canPartitionKSubsets(std::vector<int>& nums, int k);
```

```cpp
bool canPartitionKSubsets(std::vector<int>& nums, int k) {
    long long sum = 0;
    for (int x : nums) sum += x;
    if (k <= 0 || sum % k != 0) return false;
    long long target = sum / k;
    std::sort(nums.rbegin(), nums.rend());
    if (nums.empty() || nums[0] > target) return false;
    int n = (int)nums.size();
    std::vector<char> used(n, 0);
    std::function<bool(int, long long, int)> dfs =
        [&](int bucket, long long cur, int start) -> bool {
        if (bucket == k) return true;
        if (cur == target) return dfs(bucket + 1, 0, 0);
        for (int i = start; i < n; ++i) {
            if (used[i] || cur + nums[i] > target) continue;
            used[i] = 1;
            if (dfs(bucket, cur + nums[i], i + 1)) return true;
            used[i] = 0;
            if (cur == 0) break;
        }
        return false;
    };
    return dfs(0, 0, 0);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <functional>
using std::vector;
//__USER__
int main() {
    struct T { vector<int> a; int k; bool want; };
    T tests[] = {
        {{4,3,2,3,5,2,1}, 4, true},
        {{1,2,3,4}, 3, false},
        {{1,2,3,4}, 1, true},
        {{1,1,1,1}, 4, true},
        {{2,2,2,2,3,4,5}, 4, false},
        {{1,2,3,4,5,6,7,8,9,10}, 5, true},
        {{4,4,6,2,3,8,10,2,10,7}, 4, true},
        {{2,2,2,2,3,3,3,3}, 2, true}
    };
    for (auto& t : tests) {
        vector<int> a = t.a;
        if (canPartitionKSubsets(a, t.k) != t.want) { std::puts("fail"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Each subset must sum to `total / k`, so quick divisibility and max-element checks prune impossible cases. Then build one subset at a time: keep adding unused elements that fit, and once a subset hits the target start the next. Sorting descending, the "if the current subset is still empty and this element fails, stop" rule, and the `start` index all cut the branching factor sharply. Worst case is exponential but these prunes make it fast for `n <= 16`.
