## challenge: Sum of Subarray Minimums
tags: monotonic-stack, stack, dynamic-programming
track: faang
difficulty: hard

Given an array of integers `arr`, find the sum of `min(b)` over every (contiguous) subarray `b` of `arr`. Because the answer can be large, return it modulo `10^9 + 7`.

Constraints: `1 <= arr.length <= 3 * 10^4`; `1 <= arr[i] <= 3 * 10^4`.

Example: `arr = [3,1,2,4]` → `17`. The subarrays and their minimums are `[3]=3, [1]=1, [2]=2, [4]=4, [3,1]=1, [1,2]=1, [2,4]=2, [3,1,2]=1, [1,2,4]=1, [3,1,2,4]=1`, summing to `17`. Example: `arr = [11,81,94,43,3]` → `444`.

hint: Instead of enumerating subarrays, ask for each element: in how many subarrays is it the minimum? Its contribution is value times that count.
hint: Element `i` is the minimum of subarrays whose left end lies after the previous smaller element and whose right end lies before the next smaller element.
hint: Use a monotonic stack to find, for each `i`, the distance to the previous strictly-smaller element and to the next smaller-or-equal element; multiply the two distances to count the subarrays (the strict/non-strict split avoids double counting equal values).

```cpp
// starter
#include <vector>
int sumSubarrayMins(std::vector<int>& arr);
```

```cpp
int sumSubarrayMins(std::vector<int>& arr) {
    const long long MOD = 1000000007LL;
    int n = (int)arr.size();
    std::vector<int> left(n), right(n), st;
    // left[i]: # of subarrays ending at i in which arr[i] is the min
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && arr[st.back()] > arr[i]) st.pop_back();
        left[i] = st.empty() ? (i + 1) : (i - st.back());
        st.push_back(i);
    }
    st.clear();
    // right[i]: # of subarrays starting at i in which arr[i] is the min
    for (int i = n - 1; i >= 0; --i) {
        while (!st.empty() && arr[st.back()] >= arr[i]) st.pop_back();
        right[i] = st.empty() ? (n - i) : (st.back() - i);
        st.push_back(i);
    }
    long long ans = 0;
    for (int i = 0; i < n; ++i)
        ans = (ans + (long long)arr[i] * left[i] % MOD * right[i]) % MOD;
    return (int)ans;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{3,1,2,4};        if (sumSubarrayMins(a) != 17)  { std::puts("case1"); return 1; } }
    { vector<int> a{11,81,94,43,3};  if (sumSubarrayMins(a) != 444) { std::puts("case2"); return 1; } }
    { vector<int> a{1};              if (sumSubarrayMins(a) != 1)   { std::puts("case3"); return 1; } }
    { vector<int> a{2,2,2};          if (sumSubarrayMins(a) != 12)  { std::puts("case4"); return 1; } }
    { vector<int> a{1,2,3,4};        if (sumSubarrayMins(a) != 20)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Rather than enumerate the O(n^2) subarrays, sum each element's contribution: `arr[i]` counts once for every subarray in which it is the minimum. That number equals `left[i] * right[i]`, where `left[i]` is the distance to the previous strictly smaller element and `right[i]` the distance to the next smaller-or-equal element. Two monotonic-stack passes compute these boundaries; using "strictly smaller" on one side and "smaller-or-equal" on the other assigns each subarray's minimum to exactly one element, avoiding double counting when values repeat. Accumulate `arr[i] * left[i] * right[i]` modulo `10^9 + 7` in O(n) time and O(n) space.
