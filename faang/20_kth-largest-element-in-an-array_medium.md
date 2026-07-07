## challenge: Kth Largest Element in an Array
tags: heap, quickselect, sorting
track: faang
difficulty: medium

Given an integer array `nums` and an integer `k`, return the `k`-th largest element (in sorted order, not necessarily distinct). Try to do better than a full sort using a size-`k` min-heap.

Constraints: `1 <= k <= nums.length <= 10^5`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [3,2,1,5,6,4], k = 2` → `5`. Example: `nums = [3,2,3,1,2,4,5,5,6], k = 4` → `4`.

hint: You do not need the whole array sorted — only the boundary between the top k elements and the rest.
hint: Maintain a size-k min-heap; once you have seen everything, its smallest element is the k-th largest. (Quickselect is the O(n)-average alternative.)

```cpp
// starter
#include <vector>
int findKthLargest(std::vector<int>& nums, int k);
```

```cpp
int findKthLargest(std::vector<int>& nums, int k) {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
    for (int x : nums) {
        pq.push(x);
        if ((int)pq.size() > k) pq.pop();
    }
    return pq.top();
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <functional>
using std::vector;
//__USER__
int main() {
    { vector<int> n{3,2,1,5,6,4};         if (findKthLargest(n, 2) != 5) { std::puts("case1"); return 1; } }
    { vector<int> n{3,2,3,1,2,4,5,5,6};   if (findKthLargest(n, 4) != 4) { std::puts("case2"); return 1; } }
    { vector<int> n{1};                   if (findKthLargest(n, 1) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{7,7,7};               if (findKthLargest(n, 2) != 7) { std::puts("case4"); return 1; } }
    { vector<int> n{2,1};                 if (findKthLargest(n, 2) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Push elements into a min-heap capped at size k, evicting its smallest whenever it overflows; the heap's top is then the k-th largest. O(n log k) time, O(k) space. Quickselect achieves O(n) average time in place.
