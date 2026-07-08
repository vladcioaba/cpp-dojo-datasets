## challenge: Find K Closest Elements
tags: binary-search, array, two-pointers
track: faang
difficulty: hard

Given a sorted array `arr`, an integer `k`, and an integer `x`, return the `k` closest integers to `x`, in ascending order. Closeness ties are broken toward the smaller value: `a` is closer than `b` when `|a - x| < |b - x|`, or when the distances are equal and `a < b`.

Constraints: `1 <= k <= arr.length <= 10^4`, `arr` is sorted ascending, `-10^4 <= arr[i], x <= 10^4`.

Example: `arr = [1,2,3,4,5], k = 4, x = 3` → `[1,2,3,4]`. Example: `arr = [1,2,3,4,5], k = 4, x = -1` → `[1,2,3,4]`. Example: `arr = [1,2,3,4,5], k = 4, x = 4` → `[2,3,4,5]`.

hint: The answer is always a contiguous window of length `k` — so you only need to find its left edge.
hint: Binary search the left edge over `[0, n - k]`. Compare the two window boundaries: `x - arr[mid]` (distance to the left candidate) versus `arr[mid + k] - x` (distance to the element just past the window).
hint: When `x - arr[mid] > arr[mid + k] - x`, the window is too far left — slide it right (`lo = mid + 1`); the tie-break toward smaller values falls out of using `>` (not `>=`).

```cpp
// starter
#include <vector>
std::vector<int> findClosestElements(std::vector<int>& arr, int k, int x);
```

```cpp
std::vector<int> findClosestElements(std::vector<int>& arr, int k, int x) {
    int lo = 0, hi = (int)arr.size() - k;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (x - arr[mid] > arr[mid + k] - x) lo = mid + 1;
        else hi = mid;
    }
    return std::vector<int>(arr.begin() + lo, arr.begin() + lo + k);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
static bool eq(const vector<int>& a, const vector<int>& b) { return a == b; }
int main() {
    { vector<int> a{1,2,3,4,5};       if (!eq(findClosestElements(a, 4, 3),  {1,2,3,4})) { std::puts("case1"); return 1; } }
    { vector<int> a{1,2,3,4,5};       if (!eq(findClosestElements(a, 4, -1), {1,2,3,4})) { std::puts("case2"); return 1; } }
    { vector<int> a{1,2,3,4,5};       if (!eq(findClosestElements(a, 4, 4),  {2,3,4,5})) { std::puts("case3"); return 1; } }
    { vector<int> a{1,1,1,10,10,10};  if (!eq(findClosestElements(a, 1, 9),  {10}))      { std::puts("case4"); return 1; } }
    { vector<int> a{1};               if (!eq(findClosestElements(a, 1, 1),  {1}))       { std::puts("case5"); return 1; } }
    { vector<int> a{2,3,5,8,9};       if (!eq(findClosestElements(a, 3, 6),  {3,5,8}))   { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Since `arr` is sorted, the `k` closest elements form one contiguous block, so the task reduces to finding that block's left index. Binary search over `[0, n - k]`, comparing the element about to be excluded on the left (`arr[mid]`) against the element just outside the window on the right (`arr[mid + k]`). Using a strict `>` keeps the window as far left as possible on ties, which implements the "prefer smaller value" rule automatically. O(log(n - k) + k) time, O(1) extra space beyond the output.
