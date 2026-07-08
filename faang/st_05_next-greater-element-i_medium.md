## challenge: Next Greater Element I
tags: monotonic-stack, stack, hash-table
track: faang
difficulty: medium

You are given two distinct-valued arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`. For each element `nums1[i]`, find the next greater element to its right within `nums2`: the first element strictly greater than `nums1[i]` that appears after it in `nums2`. Return an array `answer` of the same length as `nums1`, where `answer[i]` is that next greater element, or `-1` if it does not exist.

Constraints: `1 <= nums1.length <= nums2.length <= 1000`; `0 <= nums1[i], nums2[i] <= 10^4`; all integers in each array are distinct; every element of `nums1` also appears in `nums2`.

Example: `nums1 = [4,1,2], nums2 = [1,3,4,2]` → `[-1,3,-1]`. Example: `nums1 = [2,4], nums2 = [1,2,3,4]` → `[3,-1]`.

hint: You could scan `nums2` to the right for each query, but a single pass over `nums2` can precompute every answer.
hint: Sweep `nums2` with a monotonic decreasing stack of values still waiting for their next greater element.
hint: When a larger value arrives, it resolves every smaller value on the stack; record those answers in a hash map, then look up each `nums1` element.

```cpp
// starter
#include <vector>
std::vector<int> nextGreaterElement(std::vector<int>& nums1, std::vector<int>& nums2);
```

```cpp
std::vector<int> nextGreaterElement(std::vector<int>& nums1, std::vector<int>& nums2) {
    std::unordered_map<int, int> nge;   // value -> next greater element
    std::vector<int> st;                // values with no greater seen yet (decreasing)
    for (int x : nums2) {
        while (!st.empty() && st.back() < x) { nge[st.back()] = x; st.pop_back(); }
        st.push_back(x);
    }
    std::vector<int> res;
    res.reserve(nums1.size());
    for (int x : nums1) {
        auto it = nge.find(x);
        res.push_back(it == nge.end() ? -1 : it->second);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
using std::vector;
//__USER__
int main() {
    { vector<int> a{4,1,2}, b{1,3,4,2};
      if (nextGreaterElement(a,b) != vector<int>({-1,3,-1})) { std::puts("case1"); return 1; } }
    { vector<int> a{2,4}, b{1,2,3,4};
      if (nextGreaterElement(a,b) != vector<int>({3,-1})) { std::puts("case2"); return 1; } }
    { vector<int> a{1}, b{1};
      if (nextGreaterElement(a,b) != vector<int>({-1})) { std::puts("case3"); return 1; } }
    { vector<int> a{7,3,5}, b{9,7,3,5,1};
      if (nextGreaterElement(a,b) != vector<int>({-1,5,-1})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Precompute the next greater element for every value in `nums2` with one monotonic-stack pass: keep a stack of values that have not yet found a larger successor. When a new value is larger than the stack top, it is that top's next greater element, so pop and record it in a hash map. Any values still on the stack at the end have no greater element. Finally map each `nums1` element through the table (defaulting to `-1`). O(n + m) time and O(n) space.
