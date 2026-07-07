## challenge: Product of Array Except Self
tags: array, prefix-sum
track: faang
difficulty: medium

Given an integer array `nums`, return an array `answer` such that `answer[i]` equals the product of all elements of `nums` except `nums[i]`. Solve it in O(n) time without using the division operator.

Constraints: `2 <= nums.length <= 10^5`, `-30 <= nums[i] <= 30`, and the product of any prefix or suffix fits in a 32-bit integer.

Example: `nums = [1,2,3,4]` → `[24,12,8,6]`. Example: `nums = [-1,1,0,-3,3]` → `[0,0,9,0,0]` (a single zero makes every other entry zero).

hint: `answer[i]` is the product of everything to the left of `i` times the product of everything to the right of `i`.
hint: Build prefix products in one left-to-right pass, then multiply by suffix products in a right-to-left pass.
hint: Store the prefix products directly in `answer`, then sweep from the right keeping a running suffix product in a single variable — no division, O(1) extra space.

```cpp
// starter
#include <vector>
std::vector<int> productExceptSelf(std::vector<int>& nums);
```

```cpp
std::vector<int> productExceptSelf(std::vector<int>& nums) {
    int n = (int)nums.size();
    std::vector<int> answer(n, 1);
    for (int i = 1; i < n; ++i) answer[i] = answer[i - 1] * nums[i - 1];
    int suffix = 1;
    for (int i = n - 1; i >= 0; --i) {
        answer[i] *= suffix;
        suffix *= nums[i];
    }
    return answer;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,3,4};      auto r = productExceptSelf(n); vector<int> w{24,12,8,6};
      if (r != w) { std::puts("case1"); return 1; } }
    { vector<int> n{-1,1,0,-3,3};  auto r = productExceptSelf(n); vector<int> w{0,0,9,0,0};
      if (r != w) { std::puts("case2"); return 1; } }
    { vector<int> n{0,0,4};        auto r = productExceptSelf(n); vector<int> w{0,0,0};
      if (r != w) { std::puts("case3"); return 1; } }
    { vector<int> n{2,3};          auto r = productExceptSelf(n); vector<int> w{3,2};
      if (r != w) { std::puts("case4"); return 1; } }
    { vector<int> n{-2,-3,-4};     auto r = productExceptSelf(n); vector<int> w{12,8,6};
      if (r != w) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Each output is the product of a prefix (everything before `i`) and a suffix (everything after `i`). First pass fills `answer[i]` with the prefix product; the second pass multiplies in the suffix product carried in a single running variable. This avoids division entirely, handles zeros naturally, and runs in O(n) time with O(1) extra space beyond the output.
