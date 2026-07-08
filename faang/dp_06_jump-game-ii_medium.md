## challenge: Jump Game II
tags: dynamic-programming, greedy
track: faang
difficulty: medium

Given an array `nums` where `nums[i]` is the maximum forward jump length from index `i`, return the minimum number of jumps needed to reach the last index starting from index `0`. The test data guarantees the last index is reachable.

Constraints: `1 <= nums.length <= 10^4`, `0 <= nums[i] <= 1000`.

Example: `nums = [2,3,1,1,4]` → `2` (jump `1` step to index `1`, then `3` steps to the end). Example: `nums = [2,3,0,1,4]` → `2`.

hint: Think in layers: with `k` jumps you can reach every index inside some prefix; the next jump extends that prefix as far as possible.
hint: Track the farthest index reachable so far and the boundary of the current jump layer.
hint: When your scan reaches the current boundary, you must spend a jump and move the boundary out to the farthest index seen.

```cpp
// starter
#include <vector>
int jump(std::vector<int>& nums);
```

```cpp
int jump(std::vector<int>& nums) {
    int n = (int)nums.size();
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < n - 1; ++i) {
        farthest = std::max(farthest, i + nums[i]);
        if (i == curEnd) {
            ++jumps;
            curEnd = farthest;
        }
    }
    return jumps;
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
    { vector<int> n{2,3,1,1,4}; if (jump(n) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{2,3,0,1,4}; if (jump(n) != 2) { std::puts("case2"); return 1; } }
    { vector<int> n{0};         if (jump(n) != 0) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2};       if (jump(n) != 1) { std::puts("case4"); return 1; } }
    { vector<int> n{1,1,1,1};   if (jump(n) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is a BFS over index layers collapsed into a linear greedy scan. As you sweep, keep the farthest index any position in the current layer can reach; when the scan hits the layer boundary you spend one jump and push the boundary out to that farthest reach. Each index is visited once for O(n) time and O(1) space.
