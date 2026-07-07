## challenge: Two Sum II - Input Array Is Sorted
tags: two-pointers, binary-search, array
track: faang
difficulty: medium

Given a 1-indexed array of integers `numbers` sorted in non-decreasing order, find two numbers that add up to `target` and return their 1-based indices `[i, j]` with `i < j`. There is exactly one solution, and you may not use the same element twice.

Constraints: `2 <= numbers.length <= 3 * 10^4`, `-1000 <= numbers[i] <= 1000`, `numbers` is sorted ascending, `-1000 <= target <= 1000`.

Example: `numbers = [2,7,11,15], target = 9` → `[1,2]`. Example: `numbers = [2,3,4], target = 6` → `[1,3]`. Example: `numbers = [-5,-2,0,3,8], target = -2` → `[1,4]`.

hint: The array is sorted — exploit that ordering instead of a hash map.
hint: Put one pointer at the start and one at the end; the current pair sum tells you which way to move.
hint: If the sum is too large move the right pointer left; if too small move the left pointer right; equal means you found the answer (convert to 1-based).

```cpp
// starter
#include <vector>
std::vector<int> twoSumSorted(std::vector<int>& numbers, int target);
```

```cpp
std::vector<int> twoSumSorted(std::vector<int>& numbers, int target) {
    int lo = 0, hi = (int)numbers.size() - 1;
    while (lo < hi) {
        int sum = numbers[lo] + numbers[hi];
        if (sum == target) return {lo + 1, hi + 1};
        if (sum < target) ++lo;
        else --hi;
    }
    return {};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,7,11,15};    auto r = twoSumSorted(n, 9);  vector<int> w{1,2};
      if (r != w) { std::puts("case1"); return 1; } }
    { vector<int> n{2,3,4};        auto r = twoSumSorted(n, 6);  vector<int> w{1,3};
      if (r != w) { std::puts("case2"); return 1; } }
    { vector<int> n{-1,0};         auto r = twoSumSorted(n, -1); vector<int> w{1,2};
      if (r != w) { std::puts("case3"); return 1; } }
    { vector<int> n{-5,-2,0,3,8};  auto r = twoSumSorted(n, -2); vector<int> w{1,4};
      if (r != w) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3,4,4,9};  auto r = twoSumSorted(n, 8);  vector<int> w{4,5};
      if (r != w) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because the input is sorted, place pointers at both ends and inspect their sum: too small means the smallest element is too weak, so advance the left pointer; too large means the largest element is too strong, so retreat the right pointer; equality yields the answer. Each step discards one candidate, giving O(n) time and O(1) space without any extra map. Convert the found 0-based indices to 1-based on return.
