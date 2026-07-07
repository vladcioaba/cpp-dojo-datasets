## challenge: Jump Game
tags: greedy, array, dynamic-programming
track: faang
difficulty: medium

Given an array `nums` where `nums[i]` is the maximum jump length from index `i`, starting at index `0`, return `true` if you can reach the last index.

Constraints: `1 <= nums.length <= 10^4`, `0 <= nums[i] <= 10^5`.

Example: `nums = [2,3,1,1,4]` → `true`. Example: `nums = [3,2,1,0,4]` → `false` (you get stuck at index 3). Example: `nums = [0]` → `true`.

```cpp
// starter
#include <vector>
bool canJump(std::vector<int>& nums);
```

```cpp
bool canJump(std::vector<int>& nums) {
    int reach = 0;
    for (int i = 0; i < (int)nums.size(); ++i) {
        if (i > reach) return false;
        reach = std::max(reach, i + nums[i]);
    }
    return true;
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
    { vector<int> n{2,3,1,1,4}; if (!canJump(n)) { std::puts("case1"); return 1; } }
    { vector<int> n{3,2,1,0,4}; if ( canJump(n)) { std::puts("case2"); return 1; } }
    { vector<int> n{0};         if (!canJump(n)) { std::puts("case3"); return 1; } }
    { vector<int> n{2,0,0};     if (!canJump(n)) { std::puts("case4"); return 1; } }
    { vector<int> n{1,0,1,0};   if ( canJump(n)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```
