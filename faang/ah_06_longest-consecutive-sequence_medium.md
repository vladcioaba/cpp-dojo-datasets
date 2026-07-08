## challenge: Longest Consecutive Sequence
tags: array, hash-table, union-find, arrays-hashing
track: faang
difficulty: medium

Given an unsorted integer array `nums`, return the length of the longest run of consecutive integers (values differing by one) that can be formed from its elements. The elements need not be adjacent in the array. Your algorithm must run in O(n) time.

Constraints: `0 <= nums.length <= 10^5`, `-10^9 <= nums[i] <= 10^9`. Values may repeat.

Example: `nums = [100,4,200,1,3,2]` → `4` (the run `1,2,3,4`). Example: `nums = [0,3,7,2,5,8,4,6,0,1]` → `9` (the run `0..8`).

hint: Sorting would work but costs O(n log n); a hash set lets you test membership of neighboring values in O(1).
hint: Only start counting a run from a value `x` when `x - 1` is absent — that value is the true left end of its streak.
hint: From each streak start, walk upward `x, x+1, x+2, ...` while the set contains the next value, and record the longest walk.

```cpp
// starter
#include <vector>
int longestConsecutive(std::vector<int>& nums);
```

```cpp
int longestConsecutive(std::vector<int>& nums) {
    std::unordered_set<int> present(nums.begin(), nums.end());
    int best = 0;
    for (int x : present) {
        if (present.count(x - 1)) continue;  // not a streak start
        int current = x, length = 1;
        while (present.count(current + 1)) { ++current; ++length; }
        best = std::max(best, length);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_set>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{100,4,200,1,3,2};            if (longestConsecutive(n) != 4) { std::puts("case1"); return 1; } }
    { vector<int> n{0,3,7,2,5,8,4,6,0,1};        if (longestConsecutive(n) != 9) { std::puts("case2"); return 1; } }
    { vector<int> n{};                           if (longestConsecutive(n) != 0) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,0,1};                    if (longestConsecutive(n) != 3) { std::puts("case4"); return 1; } }
    { vector<int> n{-3,-2,-1,5,7};               if (longestConsecutive(n) != 3) { std::puts("case5"); return 1; } }
    { vector<int> n{9};                          if (longestConsecutive(n) != 1) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Load every value into a hash set for O(1) membership tests. A number begins a consecutive run only if its predecessor is missing, so from each such start we walk upward as far as the set allows and track the maximum length. Each value is visited at most twice (once as a candidate, once during a walk), giving O(n) time and O(n) space.
