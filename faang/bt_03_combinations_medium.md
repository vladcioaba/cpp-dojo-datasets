## challenge: Combinations
tags: backtracking, combinatorics
track: faang
difficulty: medium

Given two integers `n` and `k`, return all possible combinations of `k` numbers chosen from the range `[1, n]`. Two combinations are the same if they contain the same set of numbers regardless of order, so each combination appears once. Return the answer in any order.

Constraints: `1 <= n <= 20`, `1 <= k <= n`.

Example: `n = 4, k = 2` -> `[[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]`. Example: `n = 1, k = 1` -> `[[1]]`. Example: `n = 5, k = 3` -> the 10 combinations `[[1,2,3],[1,2,4],[1,2,5],[1,3,4],[1,3,5],[1,4,5],[2,3,4],[2,3,5],[2,4,5],[3,4,5]]`.

hint: To avoid producing the same set in different orders, always pick numbers in increasing order.
hint: Backtrack with a `start` index: at each step choose some value `>= start`, append it, recurse from the next value, then undo.
hint: Stop and record when the current selection has exactly `k` numbers; you can prune early when not enough numbers remain to reach `k`.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> combine(int n, int k);
```

```cpp
std::vector<std::vector<int>> combine(int n, int k) {
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(int)> dfs = [&](int start) {
        if ((int)cur.size() == k) { res.push_back(cur); return; }
        // need (k - cur.size()) more; last usable start is n - need + 1
        int need = k - (int)cur.size();
        for (int i = start; i <= n - need + 1; ++i) {
            cur.push_back(i);
            dfs(i + 1);
            cur.pop_back();
        }
    };
    dfs(1);
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
static vector<vector<int>> reference(int n, int k) {
    vector<vector<int>> out;
    for (int mask = 0; mask < (1 << n); ++mask) {
        if (__builtin_popcount(mask) != k) continue;
        vector<int> c;
        for (int i = 0; i < n; ++i) if (mask & (1 << i)) c.push_back(i + 1);
        out.push_back(c);
    }
    return out;
}
//__USER__
int main() {
    {
        if (canon(combine(4, 2)) != canon(reference(4, 2))) { std::puts("case1"); return 1; }
    }
    {
        if (canon(combine(1, 1)) != canon(reference(1, 1))) { std::puts("case2"); return 1; }
    }
    {
        auto got = combine(5, 3);
        if (got.size() != 10) { std::puts("size3"); return 1; }
        if (canon(got) != canon(reference(5, 3))) { std::puts("case3"); return 1; }
    }
    {
        auto got = combine(6, 6);
        if (got.size() != 1) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Backtracking picks numbers in strictly increasing order by carrying a `start` index, which guarantees every set is generated exactly once. When the selection reaches size `k` it is recorded. The bound `i <= n - need + 1` prunes branches that cannot possibly gather enough remaining numbers. There are `C(n, k)` combinations, so time is O(k * C(n, k)) with O(k) recursion depth.
