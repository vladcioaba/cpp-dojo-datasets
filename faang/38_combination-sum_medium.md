## challenge: Combination Sum
tags: backtracking, array
track: faang
difficulty: medium

Given an array of `distinct` integers `candidates` and a target integer `target`, return all unique combinations of `candidates` where the chosen numbers sum to `target`. The same number may be chosen from `candidates` an unlimited number of times. Two combinations are the same if they contain the same multiset of numbers, regardless of order. Return the combinations in any order.

Constraints: `1 <= candidates.length <= 30`, `2 <= candidates[i] <= 40`, all elements distinct, `1 <= target <= 40`.

Example: `candidates = [2,3,6,7], target = 7` -> `[[2,2,3],[7]]`. Example: `candidates = [2,3,5], target = 8` -> `[[2,2,2,2],[2,3,3],[3,5]]`. Example: `candidates = [2], target = 1` -> `[]`.

hint: This is an unbounded selection problem — the same candidate can appear multiple times in one combination.
hint: Backtrack while carrying the remaining amount; to avoid duplicate multisets, only ever move forward (or stay) in the candidates index, never revisit earlier candidates.
hint: At index `i`, recurse into the same index `i` (reuse allowed) after choosing it; prune when the remaining amount drops below zero.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> combinationSum(std::vector<int>& candidates, int target);
```

```cpp
std::vector<std::vector<int>> combinationSum(std::vector<int>& candidates, int target) {
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(int, int)> dfs = [&](int start, int remain) {
        if (remain == 0) { res.push_back(cur); return; }
        for (int i = start; i < (int)candidates.size(); ++i) {
            if (candidates[i] > remain) continue;
            cur.push_back(candidates[i]);
            dfs(i, remain - candidates[i]);   // i, not i+1: reuse allowed
            cur.pop_back();
        }
    };
    dfs(0, target);
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
//__USER__
int main() {
    {
        vector<int> c{2,3,6,7};
        vector<vector<int>> want = {{2,2,3},{7}};
        if (canon(combinationSum(c, 7)) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        vector<int> c{2,3,5};
        vector<vector<int>> want = {{2,2,2,2},{2,3,3},{3,5}};
        if (canon(combinationSum(c, 8)) != canon(want)) { std::puts("case2"); return 1; }
    }
    {
        vector<int> c{2};
        auto r = combinationSum(c, 1);
        if (!r.empty()) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Backtrack over the candidates, keeping the remaining target; passing the current index (not index+1) into the recursion lets a candidate be reused, and never going backwards prevents permutations of the same multiset. Pruning when a candidate exceeds the remainder bounds the search. Time is roughly O(n^(target/min)) in the worst case, with O(target/min) recursion depth.
