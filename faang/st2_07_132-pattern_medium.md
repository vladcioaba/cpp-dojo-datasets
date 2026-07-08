## challenge: 132 Pattern
tags: monotonic-stack, stack, array
track: faang
difficulty: medium

Given an array of `n` integers `nums`, a **132 pattern** is a subsequence `nums[i]`, `nums[j]`, `nums[k]` with `i < j < k` such that `nums[i] < nums[k] < nums[j]`. Return `true` if such a pattern exists in `nums`, and `false` otherwise.

Constraints: `n == nums.length`; `1 <= n <= 2 * 10^5`; `-10^9 <= nums[i] <= 10^9`.

Example: `nums = [1,2,3,4]` → `false` (no such subsequence). Example: `nums = [3,1,4,2]` → `true` (`1, 4, 2` satisfies `1 < 2 < 4`).

hint: Fix the role of the "2" (the value `nums[k]`) and try to make it as large as possible while still below some earlier "3".
hint: Scan from right to left, keeping a stack of candidates for the "3" (the `nums[j]` value).
hint: Popped values become the best possible "2"; if any later element to the left is smaller than that "2", you have found a pattern.

```cpp
// starter
#include <vector>
bool find132pattern(std::vector<int>& nums);
```

```cpp
bool find132pattern(std::vector<int>& nums) {
    int n = (int)nums.size();
    long third = LONG_MIN;   // best "2": largest value known to sit below some later "3"
    std::vector<int> st;     // candidates for "3", values decreasing
    for (int i = n - 1; i >= 0; --i) {
        if (nums[i] < third) return true;          // nums[i] is the "1"
        while (!st.empty() && st.back() < nums[i]) {
            third = st.back();                     // nums[i] plays "3", popped value plays "2"
            st.pop_back();
        }
        st.push_back(nums[i]);
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <climits>
using std::vector;
//__USER__
int main() {
    { vector<int> a{1,2,3,4};      if (find132pattern(a))  { std::puts("case1"); return 1; } }
    { vector<int> a{3,1,4,2};      if (!find132pattern(a)) { std::puts("case2"); return 1; } }
    { vector<int> a{-1,3,2,0};     if (!find132pattern(a)) { std::puts("case3"); return 1; } }
    { vector<int> a{1,2};          if (find132pattern(a))  { std::puts("case4"); return 1; } }
    { vector<int> a{3,5,0,3,4};    if (!find132pattern(a)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Iterate from right to left. Maintain a decreasing stack of candidate "3" values and a scalar `third` holding the largest value that is known to have some larger element after it — the best available "2". When the current element is larger than the stack top, the top can serve as a "2" beneath it, so pop it into `third`. If any element encountered is strictly smaller than `third`, it is a valid "1" and a 132 pattern exists. Each element is pushed and popped at most once, so the search runs in O(n) time and O(n) space.
