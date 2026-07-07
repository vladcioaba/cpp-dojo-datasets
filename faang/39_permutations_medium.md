## challenge: Permutations
tags: backtracking, array
track: faang
difficulty: medium

Given an array `nums` of distinct integers, return all the possible permutations. You can return the answer in any order.

Constraints: `1 <= nums.length <= 6`, `-10 <= nums[i] <= 10`, all integers unique.

Example: `nums = [1,2,3]` -> the 6 permutations `[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`. Example: `nums = [0,1]` -> `[[0,1],[1,0]]`. Example: `nums = [1]` -> `[[1]]`.

hint: Every permutation fixes one element at each position, drawn from the not-yet-used elements.
hint: Backtrack: track which indices are already used, append an unused one, recurse, then undo the choice.
hint: When the current arrangement reaches length `n`, record a copy of it as a completed permutation.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> permute(std::vector<int>& nums);
```

```cpp
std::vector<std::vector<int>> permute(std::vector<int>& nums) {
    std::vector<std::vector<int>> res;
    int n = (int)nums.size();
    std::vector<int> cur;
    std::vector<char> used(n, 0);
    std::function<void()> dfs = [&]() {
        if ((int)cur.size() == n) { res.push_back(cur); return; }
        for (int i = 0; i < n; ++i) {
            if (used[i]) continue;
            used[i] = 1;
            cur.push_back(nums[i]);
            dfs();
            cur.pop_back();
            used[i] = 0;
        }
    };
    dfs();
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
#include <algorithm>
using std::vector;
static vector<vector<int>> reference(vector<int> nums) {
    std::sort(nums.begin(), nums.end());
    vector<vector<int>> out;
    do { out.push_back(nums); } while (std::next_permutation(nums.begin(), nums.end()));
    return out;
}
static long long factorial(int n) { long long f = 1; for (int i = 2; i <= n; ++i) f *= i; return f; }
//__USER__
int main() {
    {
        vector<int> n{1,2,3};
        auto got = permute(n);
        if ((long long)got.size() != factorial(3)) { std::puts("size1"); return 1; }
        std::sort(got.begin(), got.end());
        if (got != reference(n)) { std::puts("case1"); return 1; }
    }
    {
        vector<int> n{0,1};
        auto got = permute(n);
        if ((long long)got.size() != factorial(2)) { std::puts("size2"); return 1; }
        std::sort(got.begin(), got.end());
        if (got != reference(n)) { std::puts("case2"); return 1; }
    }
    {
        vector<int> n{1};
        auto got = permute(n);
        if ((long long)got.size() != factorial(1)) { std::puts("size3"); return 1; }
        std::sort(got.begin(), got.end());
        if (got != reference(n)) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Backtracking builds each permutation position by position, marking indices used as they are placed and unmarking them on the way back up the recursion tree. There are n! permutations, each of length n. O(n * n!) time to generate and copy them, O(n) recursion depth plus the used array.
