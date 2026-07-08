## challenge: Next Greater Element II
tags: monotonic-stack, stack, array
track: faang
difficulty: medium

Given a circular integer array `nums` (the element after `nums[n-1]` is `nums[0]`), return an array `answer` where `answer[i]` is the next greater number for `nums[i]`: the first value strictly greater than `nums[i]` encountered when traversing the array in order, wrapping around if necessary. If no such number exists, `answer[i]` is `-1`.

Constraints: `1 <= nums.length <= 10^4`; `-10^9 <= nums[i] <= 10^9`.

Example: `nums = [1,2,1]` → `[2,-1,2]` (the last `1` wraps around to find the `2`). Example: `nums = [1,2,3,4,3]` → `[2,3,4,-1,4]`.

hint: Handle the circularity by iterating over `2 * n` positions, indexing with `i % n`.
hint: Keep a monotonic decreasing stack of *indices* whose next greater element is still unknown.
hint: When a value larger than the stack top's value arrives, it resolves that index; only push indices during the first `n` iterations.

```cpp
// starter
#include <vector>
std::vector<int> nextGreaterElements(std::vector<int>& nums);
```

```cpp
std::vector<int> nextGreaterElements(std::vector<int>& nums) {
    int n = (int)nums.size();
    std::vector<int> res(n, -1);
    std::vector<int> st;   // indices, values decreasing
    for (int i = 0; i < 2 * n; ++i) {
        int x = nums[i % n];
        while (!st.empty() && nums[st.back()] < x) {
            res[st.back()] = x;
            st.pop_back();
        }
        if (i < n) st.push_back(i);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{1,2,1};       if (nextGreaterElements(a) != vector<int>({2,-1,2}))     { std::puts("case1"); return 1; } }
    { vector<int> a{1,2,3,4,3};   if (nextGreaterElements(a) != vector<int>({2,3,4,-1,4})) { std::puts("case2"); return 1; } }
    { vector<int> a{5,4,3,2,1};   if (nextGreaterElements(a) != vector<int>({-1,5,5,5,5})) { std::puts("case3"); return 1; } }
    { vector<int> a{1,1,1};       if (nextGreaterElements(a) != vector<int>({-1,-1,-1}))   { std::puts("case4"); return 1; } }
    { vector<int> a{100};         if (nextGreaterElements(a) != vector<int>({-1}))         { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Simulate the circular sweep by looping `i` from `0` to `2n - 1` and reading `nums[i % n]`. Maintain a stack of indices whose next greater element has not been found, keeping the corresponding values in decreasing order. Whenever the current value exceeds the value at the stack's top index, that top has found its answer, so record it and pop; repeat. Indices are only pushed during the first `n` iterations, so the second pass just resolves wrap-around cases. Each index is pushed and popped once, giving O(n) time and O(n) space.
