## challenge: Subsets
tags: backtracking, bit-manipulation, array
track: faang
difficulty: medium

Given an array `nums` of unique integers, return all possible subsets (the power set). The solution must not contain duplicate subsets. Return the subsets in any order.

Constraints: `1 <= nums.length <= 10`, `-10 <= nums[i] <= 10`, all elements unique.

Example: `nums = [1,2,3]` → `[[],[1],[2],[3],[1,2],[1,3],[2,3],[1,2,3]]` (8 subsets). Example: `nums = [0]` → `[[],[0]]`.

hint: For each element you make a binary choice — include it or not — and the leaves of that decision tree are the subsets.
hint: Backtracking over the include/exclude choices (or map each of the 2^n bitmasks to a subset).

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> subsets(std::vector<int>& nums);
```

```cpp
std::vector<std::vector<int>> subsets(std::vector<int>& nums) {
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    int n = (int)nums.size();
    std::function<void(int)> dfs = [&](int i) {
        if (i == n) { res.push_back(cur); return; }
        dfs(i + 1);                  // skip nums[i]
        cur.push_back(nums[i]);      // take nums[i]
        dfs(i + 1);
        cur.pop_back();
    };
    dfs(0);
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
static vector<vector<int>> canon(vector<vector<int>> g) {
    for (auto& row : g) std::sort(row.begin(), row.end());
    std::sort(g.begin(), g.end());
    return g;
}
static vector<vector<int>> reference(const vector<int>& nums) {
    vector<vector<int>> out;
    int n = (int)nums.size();
    for (int mask = 0; mask < (1 << n); ++mask) {
        vector<int> sub;
        for (int i = 0; i < n; ++i) if (mask & (1 << i)) sub.push_back(nums[i]);
        out.push_back(sub);
    }
    return out;
}
//__USER__
int main() {
    { vector<int> n{1,2,3};
      if (canon(subsets(n)) != canon(reference(n))) { std::puts("case1"); return 1; } }
    { vector<int> n{0};
      if (canon(subsets(n)) != canon(reference(n))) { std::puts("case2"); return 1; } }
    { vector<int> n{5,-1,4,2};
      auto got = subsets(n);
      if (got.size() != 16) { std::puts("size"); return 1; }
      if (canon(got) != canon(reference(n))) { std::puts("case3"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Backtracking explores the include-or-exclude decision for each element and records the current selection at each leaf. There are 2^n subsets, each up to length n. O(n * 2^n) time, O(n) recursion depth.
