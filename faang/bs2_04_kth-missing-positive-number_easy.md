## challenge: Kth Missing Positive Number
tags: binary-search, array
track: faang
difficulty: easy

Given a strictly increasing array `arr` of positive integers and an integer `k`, return the `k`-th positive integer that is **missing** from `arr`.

Constraints: `1 <= arr.length <= 1000`, `1 <= arr[i] <= 1000`, `arr` is strictly increasing, `1 <= k <= 1000`.

Example: `arr = [2,3,4,7,11], k = 5` → `9` (missing are `1,5,6,8,9,...`). Example: `arr = [1,2,3,4], k = 2` → `6`. Example: `arr = [5,6,7,8,9], k = 9` → `14`.

hint: At index `i`, the count of missing positives at or before `arr[i]` is `arr[i] - (i + 1)` — a non-decreasing function of `i`.
hint: Binary search for the first index where that missing count reaches `k`; everything before it has fewer than `k` gaps.
hint: Once you know how many array elements sit before the answer (`lo`), the `k`-th missing value is simply `lo + k`.

```cpp
// starter
#include <vector>
int findKthPositive(std::vector<int>& arr, int k);
```

```cpp
int findKthPositive(std::vector<int>& arr, int k) {
    int lo = 0, hi = (int)arr.size();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] - (mid + 1) >= k) hi = mid;   // enough missing by here
        else lo = mid + 1;
    }
    return lo + k;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{2,3,4,7,11}; if (findKthPositive(a, 5) != 9)  { std::puts("case1"); return 1; } }
    { vector<int> a{1,2,3,4};    if (findKthPositive(a, 2) != 6)  { std::puts("case2"); return 1; } }
    { vector<int> a{5,6,7,8,9};  if (findKthPositive(a, 9) != 14) { std::puts("case3"); return 1; } }
    { vector<int> a{2};          if (findKthPositive(a, 1) != 1)  { std::puts("case4"); return 1; } }
    { vector<int> a{1};          if (findKthPositive(a, 1) != 2)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** If no numbers were missing, `arr[i]` would equal `i + 1`; the surplus `arr[i] - (i + 1)` is exactly how many positives are missing up to `arr[i]`, and it never decreases. Binary search for the first index `lo` where this count is at least `k`. Before `lo` there are `lo` present numbers and fewer than `k` gaps, so the `k`-th missing value is `lo + k`. O(log n) time, O(1) space.
