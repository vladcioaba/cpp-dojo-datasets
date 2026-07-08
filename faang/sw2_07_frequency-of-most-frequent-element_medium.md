## challenge: Frequency of the Most Frequent Element
tags: array, sorting, sliding-window, greedy
track: faang
difficulty: medium

The frequency of an element is the number of times it occurs in an array. Given an integer array `nums` and an integer `k`, you may perform up to `k` operations, where each operation increments any single element by `1`. Return the maximum possible frequency of any one value after performing at most `k` operations.

Constraints: `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^5`, `1 <= k <= 10^5`.

Example: `nums = [1,2,4], k = 5` → `3` (raise `1` to `4` using `3` ops and `2` to `4` using `2` ops, making all three equal). Example: `nums = [1,4,8,13], k = 5` → `2`. Example: `nums = [3,9,6], k = 2` → `1`.

hint: Sort the values. To make a group of elements equal it is always cheapest to raise the smaller ones up to the largest in the group.

hint: For a sorted window ending at index `right` with target value `nums[right]`, the cost to equalize is `nums[right] * windowLength - windowSum`.

hint: Slide the window: extend right, and while the equalization cost exceeds `k`, drop the leftmost element. The widest affordable window is the answer.

```cpp
// starter
#include <vector>
int maxFrequency(std::vector<int>& nums, int k);
```

```cpp
int maxFrequency(std::vector<int>& nums, int k) {
    std::sort(nums.begin(), nums.end());
    long long sum = 0;
    int left = 0, best = 1;
    for (int right = 0; right < (int)nums.size(); ++right) {
        sum += nums[right];
        while ((long long)nums[right] * (right - left + 1) - sum > k) {
            sum -= nums[left++];
        }
        best = std::max(best, right - left + 1);
    }
    return best;
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
    { vector<int> n{1,2,4}; if (maxFrequency(n,5)!=3) { std::puts("case1"); return 1; } }
    { vector<int> n{1,4,8,13}; if (maxFrequency(n,5)!=2) { std::puts("case2"); return 1; } }
    { vector<int> n{3,9,6}; if (maxFrequency(n,2)!=1) { std::puts("case3"); return 1; } }
    { vector<int> n{9,9,9}; if (maxFrequency(n,1)!=3) { std::puts("case4"); return 1; } }
    { vector<int> n{1,1,1,2,2,4}; if (maxFrequency(n,5)!=5) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** After sorting, the cheapest target for any contiguous group is its largest member, so equalizing a window `[left, right]` to `nums[right]` costs `nums[right] * (right - left + 1) - windowSum`. Slide a window across the sorted array: as the right edge advances, add to the running sum; while the cost to lift the whole window to `nums[right]` exceeds the operation budget `k`, discard the leftmost element. The maximum window width reached is the best achievable frequency. Sorting is O(n log n), the two-pointer pass is O(n), and a 64-bit sum prevents overflow.
