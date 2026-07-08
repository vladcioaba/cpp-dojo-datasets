## challenge: Combination Sum III
tags: backtracking, array
track: faang
difficulty: medium

Find all valid combinations of `k` distinct numbers chosen from `1` to `9` that add up to `n`. Each number `1`–`9` may be used at most once, and every combination must use exactly `k` numbers. Return the list of combinations in any order; the numbers within a combination may appear in any order.

Constraints: `2 <= k <= 9`, `1 <= n <= 60`.

Example: `k = 3, n = 7` -> `[[1,2,4]]`. Example: `k = 3, n = 9` -> `[[1,2,6],[1,3,5],[2,3,4]]`. Example: `k = 4, n = 1` -> `[]` (four distinct positive numbers cannot sum to 1).

hint: The pool is the fixed set `1`–`9`; choose numbers in strictly increasing order so each multiset is produced only once.
hint: Carry the remaining sum and the next candidate to consider; stop a branch once you already hold `k` numbers or the next candidate exceeds the remainder.
hint: Because candidates only increase, a candidate greater than the remaining sum lets you break the loop early.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> combinationSum3(int k, int n);
```

```cpp
std::vector<std::vector<int>> combinationSum3(int k, int n) {
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(int, int)> dfs = [&](int start, int remain) {
        if ((int)cur.size() == k) {
            if (remain == 0) res.push_back(cur);
            return;
        }
        for (int v = start; v <= 9; ++v) {
            if (v > remain) break;
            cur.push_back(v);
            dfs(v + 1, remain - v);
            cur.pop_back();
        }
    };
    dfs(1, n);
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
    for (auto& r : g) std::sort(r.begin(), r.end());
    std::sort(g.begin(), g.end());
    return g;
}
static vector<vector<int>> reference(int k, int n) {
    vector<vector<int>> out;
    for (int mask = 0; mask < (1 << 9); ++mask) {
        if (__builtin_popcount(mask) != k) continue;
        int s = 0; vector<int> comb;
        for (int i = 0; i < 9; ++i)
            if (mask & (1 << i)) { comb.push_back(i + 1); s += i + 1; }
        if (s == n) out.push_back(comb);
    }
    return out;
}
//__USER__
int main() {
    int ks[] = {3, 3, 4, 2, 3, 9, 2, 5};
    int ns[] = {7, 9, 1, 18, 15, 45, 17, 25};
    for (int t = 0; t < 8; ++t) {
        if (canon(combinationSum3(ks[t], ns[t])) != canon(reference(ks[t], ns[t]))) {
            std::printf("fail k=%d n=%d\n", ks[t], ns[t]); return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** Standard subset backtracking constrained to nine candidates: pick numbers in increasing order, decrement the remaining sum, and record a combination when exactly `k` numbers reach zero. Increasing order prevents permuted duplicates, and breaking when a candidate exceeds the remainder prunes the tree. The harness confirms results against a brute-force enumeration of all `2^9` bitmasks.
