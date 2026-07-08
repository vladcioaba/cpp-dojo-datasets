## challenge: Peak Index in a Mountain Array
tags: binary-search, array
track: faang
difficulty: medium

An array `arr` is a **mountain** if it strictly increases up to a single peak and then strictly decreases. Given such an array, return the index of the peak element. You must solve it in O(log n).

Constraints: `3 <= arr.length <= 10^5`, `0 <= arr[i] <= 10^6`, `arr` is guaranteed to be a mountain array.

Example: `arr = [0,1,0]` → `1`. Example: `arr = [0,2,1,0]` → `1`. Example: `arr = [3,4,5,1]` → `2`.

hint: The comparison `arr[mid] < arr[mid + 1]` tells you which slope you are on: still ascending, or past the peak.
hint: If `arr[mid] < arr[mid + 1]` the peak is strictly to the right (`lo = mid + 1`); otherwise `mid` could be the peak (`hi = mid`).
hint: Iterate on `[0, n - 1)` with `hi = n - 1`, so `mid + 1` is always a valid index.

```cpp
// starter
#include <vector>
int peakIndexInMountainArray(std::vector<int>& arr);
```

```cpp
int peakIndexInMountainArray(std::vector<int>& arr) {
    int lo = 0, hi = (int)arr.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < arr[mid + 1]) lo = mid + 1;   // on the ascending slope
        else hi = mid;                               // at or past the peak
    }
    return lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{0,1,0};      if (peakIndexInMountainArray(a) != 1) { std::puts("case1"); return 1; } }
    { vector<int> a{0,2,1,0};    if (peakIndexInMountainArray(a) != 1) { std::puts("case2"); return 1; } }
    { vector<int> a{0,10,5,2};   if (peakIndexInMountainArray(a) != 1) { std::puts("case3"); return 1; } }
    { vector<int> a{3,4,5,1};    if (peakIndexInMountainArray(a) != 2) { std::puts("case4"); return 1; } }
    { vector<int> a{24,69,100,99,79,78,67,36,26,19};
      if (peakIndexInMountainArray(a) != 2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Compare each `mid` with its right neighbor. On the ascending slope `arr[mid] < arr[mid + 1]`, so the peak lies strictly to the right and we set `lo = mid + 1`; otherwise we are at the peak or on the descending slope, and `mid` remains a candidate (`hi = mid`). The window shrinks to the unique peak. Keeping `hi = n - 1` guarantees `mid + 1` is in bounds. O(log n) time, O(1) space.
