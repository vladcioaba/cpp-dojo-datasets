## challenge: Subsets II
tags: backtracking, bit-manipulation, array
track: faang
difficulty: medium

Given an integer array `nums` that may contain duplicates, return all possible subsets (the power set). The solution set must not contain duplicate subsets. Return the subsets in any order.

Constraints: `1 <= nums.length <= 10`, `-10 <= nums[i] <= 10`.

Example: `nums = [1,2,2]` -> `[[],[1],[1,2],[1,2,2],[2],[2,2]]` (6 subsets). Example: `nums = [0]` -> `[[],[0]]`. Example: `nums = [4,4,4,1,4]` -> the 10 distinct subsets of the multiset `{1,4,4,4,4}`.

hint: Sort so equal values are adjacent; the danger is generating the same subset from different copies of a repeated value.
hint: Backtrack with a `start` index, recording the current selection at every node (not just the leaves) — that captures all subset sizes.
hint: Within one recursion level, if `nums[i] == nums[i-1]` and `i > start`, skip `i`; picking the same value again at the same depth would duplicate a subset.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> subsetsWithDup(std::vector<int>& nums);
```

```cpp
std::vector<std::vector<int>> subsetsWithDup(std::vector<int>& nums) {
    std::sort(nums.begin(), nums.end());
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(int)> dfs = [&](int start) {
        res.push_back(cur);                       // every node is a valid subset
        for (int i = start; i < (int)nums.size(); ++i) {
            if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate at this level
            cur.push_back(nums[i]);
            dfs(i + 1);
            cur.pop_back();
        }
    };
    dfs(0);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <set>
#include <functional>
#include <algorithm>
using std::vector;
static vector<vector<int>> canon(vector<vector<int>> g) {
    for (auto& row : g) std::sort(row.begin(), row.end());
    std::sort(g.begin(), g.end());
    return g;
}
static vector<vector<int>> reference(vector<int> nums) {
    std::sort(nums.begin(), nums.end());
    int n = (int)nums.size();
    std::set<vector<int>> uniq;
    for (int mask = 0; mask < (1 << n); ++mask) {
        vector<int> sub;
        for (int i = 0; i < n; ++i) if (mask & (1 << i)) sub.push_back(nums[i]);
        uniq.insert(sub);
    }
    return vector<vector<int>>(uniq.begin(), uniq.end());
}
//__USER__
int main() {
    {
        vector<int> n{1,2,2};
        auto got = subsetsWithDup(n);
        if (got.size() != 6) { std::puts("size1"); return 1; }
        if (canon(got) != canon(reference({1,2,2}))) { std::puts("case1"); return 1; }
    }
    {
        vector<int> n{0};
        if (canon(subsetsWithDup(n)) != canon(reference({0}))) { std::puts("case2"); return 1; }
    }
    {
        vector<int> n{4,4,4,1,4};
        auto got = subsetsWithDup(n);
        auto c = canon(got);
        // no duplicate subsets in output
        auto d = c; d.erase(std::unique(d.begin(), d.end()), d.end());
        if (c != d) { std::puts("dup3"); return 1; }
        if (c != canon(reference({4,4,4,1,4}))) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Sorting brings equal values together. The backtracking records a subset at every node, and within a single recursion level it skips any value equal to its predecessor (unless it is the first pick at that level), which prevents the same multiset subset from being emitted twice. There are at most `2^n` subsets; time is O(n * 2^n) with O(n) recursion depth.
