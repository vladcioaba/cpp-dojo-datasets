## challenge: Number of Dice Rolls With Target Sum
tags: dynamic-programming, array
track: faang
difficulty: medium

You have `n` dice, and each die has `k` faces numbered from `1` to `k`. Return the number of ways to roll all the dice so that the sum of the face-up numbers equals `target`. Because the answer may be large, return it modulo `10^9 + 7`.

Constraints: `1 <= n, k <= 30`, `1 <= target <= 1000`.

Example: `n = 1, k = 6, target = 3` → `1` (only the single die showing `3`). Example: `n = 2, k = 6, target = 7` → `6` (`1+6, 2+5, 3+4, 4+3, 5+2, 6+1`).

hint: Add one die at a time; a running sum `s` can grow by any face value from `1` to `k`.
hint: Keep `dp[s]` = ways to reach sum `s` using the dice rolled so far, and roll the next die into a fresh array.

```cpp
// starter
int numRollsToTarget(int n, int k, int target);
```

```cpp
int numRollsToTarget(int n, int k, int target) {
    const long long MOD = 1000000007LL;
    std::vector<long long> dp(target + 1, 0);
    dp[0] = 1;
    for (int d = 0; d < n; ++d) {
        std::vector<long long> ndp(target + 1, 0);
        for (int s = 0; s <= target; ++s) {
            if (dp[s] == 0) continue;
            for (int f = 1; f <= k && s + f <= target; ++f)
                ndp[s + f] = (ndp[s + f] + dp[s]) % MOD;
        }
        dp.swap(ndp);
    }
    return (int)dp[target];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
//__USER__
int main() {
    if (numRollsToTarget(1, 6, 3)    != 1)         { std::puts("case1"); return 1; }
    if (numRollsToTarget(2, 6, 7)    != 6)         { std::puts("case2"); return 1; }
    if (numRollsToTarget(2, 5, 10)   != 1)         { std::puts("case3"); return 1; }
    if (numRollsToTarget(1, 2, 3)    != 0)         { std::puts("case4"); return 1; }
    if (numRollsToTarget(30, 30, 500)!= 222616187) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Let `dp[s]` count the ways the dice rolled so far sum to `s`. Adding one more die transitions each reachable `s` to `s+f` for every face `f` in `1..k`, accumulated into a fresh array modulo `10^9 + 7`; swapping the arrays advances one die. After processing all `n` dice, `dp[target]` is the answer. The work is O(n · target · k) time with O(target) space, comfortably within the given limits.
