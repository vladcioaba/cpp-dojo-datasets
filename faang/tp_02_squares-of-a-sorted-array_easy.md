## challenge: Squares of a Sorted Array
tags: two-pointers, array, sorting
track: faang
difficulty: easy

Given an integer array `nums` sorted in non-decreasing order, return an array of the squares of each number, also sorted in non-decreasing order.

Constraints: `1 <= nums.length <= 10^4`, `-10^4 <= nums[i] <= 10^4`, `nums` is sorted in non-decreasing order.

Example: `nums = [-4,-1,0,3,10]` → `[0,1,9,16,100]`. Example: `nums = [-7,-3,2,3,11]` → `[4,9,9,49,121]`.

hint: A trivial O(n log n) solution squares then sorts — but the input ordering already tells you where the largest squares live.
hint: The biggest squares come from the two ends of the array (the most negative and the most positive values), not the middle.
hint: Run two pointers from both ends; compare the absolute magnitudes, and write the larger square into the result from the back toward the front.

```cpp
// starter
#include <vector>
std::vector<int> sortedSquares(std::vector<int>& nums);
```

```cpp
std::vector<int> sortedSquares(std::vector<int>& nums) {
    int n = (int)nums.size();
    std::vector<int> res(n);
    int lo = 0, hi = n - 1;
    for (int p = n - 1; p >= 0; --p) {
        int a = nums[lo] * nums[lo];
        int b = nums[hi] * nums[hi];
        if (a > b) { res[p] = a; ++lo; }
        else { res[p] = b; --hi; }
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
    { vector<int> n{-4,-1,0,3,10}; if (sortedSquares(n) != vector<int>{0,1,9,16,100}) { std::puts("case1"); return 1; } }
    { vector<int> n{-7,-3,2,3,11}; if (sortedSquares(n) != vector<int>{4,9,9,49,121}) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3};        if (sortedSquares(n) != vector<int>{1,4,9}) { std::puts("case3"); return 1; } }
    { vector<int> n{-5,-3,-1};     if (sortedSquares(n) != vector<int>{1,9,25}) { std::puts("case4"); return 1; } }
    { vector<int> n{0};            if (sortedSquares(n) != vector<int>{0}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** After squaring, the largest values sit at the two ends of the sorted input (extreme negatives and extreme positives). Place a pointer at each end and, comparing squared magnitudes, drop the larger one into the result array filled from back to front. Each element is placed exactly once, yielding O(n) time and O(n) space for the output, beating the O(n log n) square-then-sort approach.
