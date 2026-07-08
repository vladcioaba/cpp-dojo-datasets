## challenge: Combination Sum II
tags: backtracking, array
track: faang
difficulty: medium

Given a collection of candidate numbers `candidates` (which may contain duplicates) and a target integer `target`, return all unique combinations of `candidates` where the chosen numbers sum to `target`. Each number in `candidates` may be used at most once in a combination. Two combinations are the same if they contain the same multiset of numbers regardless of order, and the answer must not contain duplicate combinations. Return the combinations in any order.

Constraints: `1 <= candidates.length <= 100`, `1 <= candidates[i] <= 50`, `1 <= target <= 30`.

Example: `candidates = [10,1,2,7,6,1,5], target = 8` -> `[[1,1,6],[1,2,5],[1,7],[2,6]]`. Example: `candidates = [2,5,2,1,2], target = 5` -> `[[1,2,2],[5]]`. Example: `candidates = [2], target = 1` -> `[]`.

hint: Sort first so that equal values sit next to each other; then a duplicate multiset is easy to skip.
hint: Backtrack with a `start` index and use each element at most once by recursing from `i + 1`, never `i`.
hint: At a given depth, if `candidates[i] == candidates[i-1]` and `i > start`, skip `i` — that value was already tried as the first pick at this level, so reusing it here would repeat a combination.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> combinationSum2(std::vector<int>& candidates, int target);
```

```cpp
std::vector<std::vector<int>> combinationSum2(std::vector<int>& candidates, int target) {
    std::sort(candidates.begin(), candidates.end());
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(int, int)> dfs = [&](int start, int remain) {
        if (remain == 0) { res.push_back(cur); return; }
        for (int i = start; i < (int)candidates.size(); ++i) {
            if (i > start && candidates[i] == candidates[i - 1]) continue; // skip duplicates at this level
            if (candidates[i] > remain) break;                            // sorted: nothing smaller ahead
            cur.push_back(candidates[i]);
            dfs(i + 1, remain - candidates[i]);                           // i+1: use each element once
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
        vector<int> c{10,1,2,7,6,1,5};
        vector<vector<int>> want = {{1,1,6},{1,2,5},{1,7},{2,6}};
        if (canon(combinationSum2(c, 8)) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        vector<int> c{2,5,2,1,2};
        vector<vector<int>> want = {{1,2,2},{5}};
        if (canon(combinationSum2(c, 5)) != canon(want)) { std::puts("case2"); return 1; }
    }
    {
        vector<int> c{2};
        if (!combinationSum2(c, 1).empty()) { std::puts("case3"); return 1; }
    }
    {
        vector<int> c{1,1,1,1,1};
        vector<vector<int>> want = {{1,1,1}};
        if (canon(combinationSum2(c, 3)) != canon(want)) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Sorting groups equal values together so duplicate combinations can be pruned with a single check: at any recursion level, skip a candidate equal to the previous one unless it is the first choice at that level. Recursing from `i + 1` enforces at-most-once use, and breaking when a sorted candidate exceeds the remaining target bounds the search. Worst-case time is O(2^n) over the candidates with O(n) recursion depth.
