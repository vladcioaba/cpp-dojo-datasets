## challenge: Rotate Array
tags: two-pointers, array, math
track: faang
difficulty: medium

Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative. Do it in place with O(1) extra space.

Constraints: `1 <= nums.length <= 10^5`, `-2^31 <= nums[i] <= 2^31 - 1`, `0 <= k <= 10^5`.

Example: `nums = [1,2,3,4,5,6,7], k = 3` → `[5,6,7,1,2,3,4]`. Example: `nums = [-1,-100,3,99], k = 2` → `[3,99,-1,-100]`.

hint: A right rotation by `k` moves the last `k` elements to the front — and `k` can be larger than the array, so reduce it modulo `n` first.
hint: Reversal is the classic O(1)-space trick: reversing the whole array puts the tail up front but in the wrong internal order.
hint: Reverse the entire array, then reverse the first `k` elements, then reverse the remaining `n - k` elements.

```cpp
// starter
#include <vector>
void rotate(std::vector<int>& nums, int k);
```

```cpp
void rotate(std::vector<int>& nums, int k) {
    int n = (int)nums.size();
    if (n == 0) return;
    k %= n;
    std::reverse(nums.begin(), nums.end());
    std::reverse(nums.begin(), nums.begin() + k);
    std::reverse(nums.begin() + k, nums.end());
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
    { vector<int> n{1,2,3,4,5,6,7}; rotate(n, 3); if (n != vector<int>{5,6,7,1,2,3,4}) { std::puts("case1"); return 1; } }
    { vector<int> n{-1,-100,3,99};  rotate(n, 2); if (n != vector<int>{3,99,-1,-100}) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2};           rotate(n, 3); if (n != vector<int>{2,1}) { std::puts("case3"); return 1; } }
    { vector<int> n{1};             rotate(n, 0); if (n != vector<int>{1}) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3};         rotate(n, 3); if (n != vector<int>{1,2,3}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Rotating right by `k` is equivalent to three reversals. First reduce `k` modulo `n` since rotating by `n` is a no-op. Reversing the whole array brings the last `k` elements to the front but reverses each block; reversing the first `k` elements and then the last `n - k` elements restores their correct internal order. Each element is touched a constant number of times, giving O(n) time and O(1) extra space.
