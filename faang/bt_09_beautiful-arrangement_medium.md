## challenge: Beautiful Arrangement
tags: backtracking, bitmask, dynamic-programming
track: faang
difficulty: medium

Suppose you have `n` integers labelled `1` through `n`. A permutation of these `n` integers is called a beautiful arrangement if, for every position `i` (1-indexed), at least one of the following holds: `perm[i]` is divisible by `i`, or `i` is divisible by `perm[i]`. Given `n`, return the number of beautiful arrangements you can construct.

Constraints: `1 <= n <= 15`.

Example: `n = 1` -> `1`. Example: `n = 2` -> `2` (both `[1,2]` and `[2,1]` are beautiful). Example: `n = 3` -> `3`.

hint: Build the arrangement one position at a time; at position `pos` only values that satisfy the divisibility rule with `pos` are candidates.
hint: Track which values are already used (a boolean array or a bitmask); when `pos` exceeds `n` you have completed one valid arrangement, so add 1.
hint: Filling positions from `1` upward and pruning invalid placements immediately prunes far more of the tree than generating full permutations and filtering.

```cpp
// starter
int countArrangement(int n);
```

```cpp
int countArrangement(int n) {
    std::vector<char> used(n + 1, 0);
    std::function<int(int)> dfs = [&](int pos) -> int {
        if (pos > n) return 1;
        int count = 0;
        for (int v = 1; v <= n; ++v) {
            if (!used[v] && (v % pos == 0 || pos % v == 0)) {
                used[v] = 1;
                count += dfs(pos + 1);
                used[v] = 0;
            }
        }
        return count;
    };
    return dfs(1);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
#include <algorithm>
using std::vector;
static int reference(int n) {
    vector<int> p(n);
    for (int i = 0; i < n; ++i) p[i] = i + 1;
    int cnt = 0;
    do {
        bool ok = true;
        for (int i = 0; i < n; ++i) {
            int pos = i + 1, v = p[i];
            if (!(v % pos == 0 || pos % v == 0)) { ok = false; break; }
        }
        if (ok) ++cnt;
    } while (std::next_permutation(p.begin(), p.end()));
    return cnt;
}
//__USER__
int main() {
    for (int n = 1; n <= 8; ++n) {
        if (countArrangement(n) != reference(n)) { std::printf("n=%d\n", n); return 1; }
    }
    // spot-check known values
    if (countArrangement(1) != 1) { std::puts("k1"); return 1; }
    if (countArrangement(2) != 2) { std::puts("k2"); return 1; }
    if (countArrangement(4) != 8) { std::puts("k4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Rather than generate all `n!` permutations and filter, place values position by position and only ever try candidates that satisfy the divisibility condition with the current position, marking them used. Completing all positions counts one arrangement. The aggressive pruning makes this vastly faster than brute force; the reference here brute-forces small `n` to confirm correctness. Time is bounded by the number of partial valid arrangements, with O(n) recursion depth.
