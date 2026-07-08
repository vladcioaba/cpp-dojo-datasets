## challenge: Permutations II
tags: backtracking, array
track: faang
difficulty: medium

Given a collection of numbers `nums` that might contain duplicates, return all possible unique permutations. Two permutations are the same if they list the same values in the same order, and the answer must not contain duplicates. Return the permutations in any order.

Constraints: `1 <= nums.length <= 8`, `-10 <= nums[i] <= 10`.

Example: `nums = [1,1,2]` -> `[[1,1,2],[1,2,1],[2,1,1]]`. Example: `nums = [1,2,3]` -> the 6 permutations of three distinct values. Example: `nums = [2,2,2]` -> `[[2,2,2]]`.

hint: Sort so equal values are adjacent, then track which positions are used as you build a permutation.
hint: The trap is choosing equal values in a swapped order that yields the same permutation; you need a rule that fixes the relative order of equal copies.
hint: Skip `nums[i]` when `nums[i] == nums[i-1]` and the previous equal copy `i-1` is not currently used — that forces equal values to be placed left to right, so each distinct permutation is generated once.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> permuteUnique(std::vector<int>& nums);
```

```cpp
std::vector<std::vector<int>> permuteUnique(std::vector<int>& nums) {
    std::sort(nums.begin(), nums.end());
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::vector<char> used(nums.size(), 0);
    std::function<void()> dfs = [&]() {
        if (cur.size() == nums.size()) { res.push_back(cur); return; }
        for (int i = 0; i < (int)nums.size(); ++i) {
            if (used[i]) continue;
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue; // fix order of equal copies
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
static vector<vector<int>> canonOuter(vector<vector<int>> g) {
    std::sort(g.begin(), g.end()); // rows are ordered permutations; only sort the outer list
    return g;
}
static vector<vector<int>> reference(vector<int> nums) {
    std::sort(nums.begin(), nums.end());
    vector<vector<int>> out;
    do { out.push_back(nums); } while (std::next_permutation(nums.begin(), nums.end()));
    return out; // already sorted and distinct
}
//__USER__
int main() {
    {
        vector<int> n{1,1,2};
        auto got = permuteUnique(n);
        if (got.size() != 3) { std::puts("size1"); return 1; }
        if (canonOuter(got) != reference({1,1,2})) { std::puts("case1"); return 1; }
    }
    {
        vector<int> n{1,2,3};
        auto got = permuteUnique(n);
        if (got.size() != 6) { std::puts("size2"); return 1; }
        if (canonOuter(got) != reference({1,2,3})) { std::puts("case2"); return 1; }
    }
    {
        vector<int> n{2,2,2};
        auto got = permuteUnique(n);
        if (got.size() != 1) { std::puts("size3"); return 1; }
        if (canonOuter(got) != reference({2,2,2})) { std::puts("case3"); return 1; }
    }
    {
        vector<int> n{3,3,0,3};
        auto got = permuteUnique(n);
        auto c = canonOuter(got);
        auto d = c; d.erase(std::unique(d.begin(), d.end()), d.end());
        if (c != d) { std::puts("dup4"); return 1; }              // no duplicate permutations
        if (c != reference({3,3,0,3})) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Sort the values, then build permutations position by position over the used flags. Equal copies are forced into a fixed left-to-right order by the guard `nums[i] == nums[i-1] && !used[i-1]`, which refuses to place a duplicate before its identical predecessor has been placed. This emits each distinct permutation exactly once. With `n` items there are at most `n!` permutations, so time is O(n * n!) and recursion depth O(n).
