## challenge: Number of Sub-arrays of Size K With Average Greater Than or Equal to Threshold
tags: array, sliding-window
track: faang
difficulty: easy

Given an integer array `arr`, an integer `k`, and an integer `threshold`, return the number of contiguous sub-arrays of length exactly `k` whose average value is greater than or equal to `threshold`.

Constraints: `1 <= arr.length <= 10^5`, `1 <= arr[i] <= 10^4`, `1 <= k <= arr.length`, `0 <= threshold <= 10^4`.

Example: `arr = [2,2,2,2,5,5,5,8], k = 3, threshold = 4` → `3` (the windows `[2,5,5]`, `[5,5,5]`, `[5,5,8]` each average at least `4`). Example: `arr = [11,13,17,23,29,31,7,5,2,3], k = 3, threshold = 5` → `6`.

hint: The average of a length-`k` window is at least `threshold` exactly when its sum is at least `k * threshold`, so you never need to divide.

hint: Compute the sum of the first `k` elements, then slide the window one step at a time, adding the entering value and subtracting the leaving one in O(1).

hint: Compare each window sum against the precomputed bound `k * threshold` and count the qualifying windows.

```cpp
// starter
#include <vector>
int numOfSubarrays(std::vector<int>& arr, int k, int threshold);
```

```cpp
int numOfSubarrays(std::vector<int>& arr, int k, int threshold) {
    long long bound = (long long)k * threshold;
    long long sum = 0;
    for (int i = 0; i < k; ++i) sum += arr[i];
    int count = sum >= bound ? 1 : 0;
    for (int i = k; i < (int)arr.size(); ++i) {
        sum += arr[i] - arr[i - k];
        if (sum >= bound) ++count;
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{2,2,2,2,5,5,5,8}; if (numOfSubarrays(a,3,4)!=3) { std::puts("case1"); return 1; } }
    { vector<int> a{11,13,17,23,29,31,7,5,2,3}; if (numOfSubarrays(a,3,5)!=6) { std::puts("case2"); return 1; } }
    { vector<int> a{7}; if (numOfSubarrays(a,1,7)!=1) { std::puts("case3"); return 1; } }
    { vector<int> a{1,1,1,1}; if (numOfSubarrays(a,2,2)!=0) { std::puts("case4"); return 1; } }
    { vector<int> a{4,4,4,4}; if (numOfSubarrays(a,4,4)!=1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A window's average meets the threshold precisely when its sum reaches `k * threshold`, so precompute that bound once and work entirely with sums. Slide a fixed-size window across the array, maintaining the running sum in O(1) per step by adding the new element and dropping the old one, and increment the counter whenever the sum clears the bound. O(n) time, O(1) space. A 64-bit bound and accumulator keep the arithmetic safe.
