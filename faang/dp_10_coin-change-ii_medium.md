## challenge: Coin Change II
tags: dynamic-programming, combinatorics
track: faang
difficulty: medium

Given coin denominations `coins` and a target `amount`, return the number of distinct combinations of coins that sum to `amount`. You have an unlimited supply of each coin, and two combinations differing only in order count as the same.

Constraints: `1 <= coins.length <= 300`, `1 <= coins[i] <= 5000`, all coins distinct, `0 <= amount <= 5000`. The answer fits in a 32-bit signed integer.

Example: `coins = [1,2,5], amount = 5` → `4` (`5`; `2+2+1`; `2+1+1+1`; `1+1+1+1+1`). Example: `coins = [2], amount = 3` → `0`.

hint: To avoid counting the same multiset in different orders, fix an outer order over coin types rather than over amounts.
hint: Let `dp[a]` be the number of ways to form amount `a`; start with `dp[0] = 1`.
hint: Process one coin at a time, and for that coin add `dp[a - coin]` into `dp[a]` for increasing `a`.

```cpp
// starter
#include <vector>
int change(int amount, std::vector<int>& coins);
```

```cpp
int change(int amount, std::vector<int>& coins) {
    std::vector<int> dp(amount + 1, 0);
    dp[0] = 1;
    for (int c : coins)
        for (int a = c; a <= amount; ++a)
            dp[a] += dp[a - c];
    return dp[amount];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> c{1,2,5};   if (change(5, c)  != 4) { std::puts("case1"); return 1; } }
    { vector<int> c{2};       if (change(3, c)  != 0) { std::puts("case2"); return 1; } }
    { vector<int> c{10};      if (change(10, c) != 1) { std::puts("case3"); return 1; } }
    { vector<int> c{7};       if (change(0, c)  != 1) { std::puts("case4"); return 1; } }
    { vector<int> c{1,2,3};   if (change(4, c)  != 4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Counting combinations rather than permutations requires the coin loop to be the outer loop, so each multiset is generated exactly once. dp[a] accumulates the number of ways to reach amount a; adding a coin means dp[a] += dp[a - coin] scanned in increasing a to allow reuse. This is O(coins * amount) time and O(amount) space.
