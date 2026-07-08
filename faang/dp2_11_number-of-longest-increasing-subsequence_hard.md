## challenge: Number of Longest Increasing Subsequence
tags: dynamic-programming, array, binary-indexed-tree
track: faang
difficulty: hard

Given an integer array `nums`, return the number of longest strictly increasing subsequences. A subsequence keeps the original order but need not be contiguous, and "longest" refers to the maximum possible length of a strictly increasing subsequence.

Constraints: `1 <= nums.length <= 2000`, `-10^6 <= nums[i] <= 10^6`. The answer is guaranteed to fit in a 32-bit signed integer.

Example: `nums = [1,3,5,4,7]` → `2` (the two longest are `[1,3,4,7]` and `[1,3,5,7]`). Example: `nums = [2,2,2,2,2]` → `5` (each single element is a longest increasing subsequence of length 1).

hint: Track two things at each index: the length of the longest increasing subsequence ending there, and how many achieve it.
hint: When extending from `j` to `i`, a strictly longer chain resets the count, while an equal-length chain adds to it.

```cpp
// starter
#include <vector>
int findNumberOfLIS(std::vector<int>& nums);
```

```cpp
int findNumberOfLIS(std::vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    std::vector<int> len(n, 1), cnt(n, 1);
    int longest = 1;
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[j] < nums[i]) {
                if (len[j] + 1 > len[i]) { len[i] = len[j] + 1; cnt[i] = cnt[j]; }
                else if (len[j] + 1 == len[i]) cnt[i] += cnt[j];
            }
        }
        longest = std::max(longest, len[i]);
    }
    int total = 0;
    for (int i = 0; i < n; ++i)
        if (len[i] == longest) total += cnt[i];
    return total;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,3,5,4,7};         if (findNumberOfLIS(n) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{2,2,2,2,2};         if (findNumberOfLIS(n) != 5) { std::puts("case2"); return 1; } }
    { vector<int> n{1};                 if (findNumberOfLIS(n) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{5,4,3,2,1};         if (findNumberOfLIS(n) != 5) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,4,3,5,4,7,2};   if (findNumberOfLIS(n) != 3) { std::puts("case5"); return 1; } }
    { vector<int> n{3,1,2};             if (findNumberOfLIS(n) != 1) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** For each index keep two arrays: `len[i]`, the length of the longest strictly increasing subsequence ending at `i`, and `cnt[i]`, how many such subsequences exist. Scanning every earlier index `j` with `nums[j] < nums[i]`: if it yields a strictly longer chain (`len[j]+1 > len[i]`), reset `len[i]` and copy `cnt[j]`; if it ties the current best, add `cnt[j]`. After computing the global maximum length, sum the counts at all indices reaching it. O(n^2) time, O(n) space.
