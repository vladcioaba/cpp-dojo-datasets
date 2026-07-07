## challenge: Coin Change
tags: dynamic-programming, bfs
track: faang
difficulty: medium

Given coin denominations `coins` and a target `amount`, return the fewest number of coins needed to make up that amount, or `-1` if it cannot be made. You have an unlimited supply of each coin.

Constraints: `1 <= coins.length <= 12`, `1 <= coins[i] <= 2^31 - 1`, `0 <= amount <= 10^4`.

Example: `coins = [1,2,5], amount = 11` → `3` (`5+5+1`). Example: `coins = [2], amount = 3` → `-1`. Example: `amount = 0` → `0`.

```cpp
// starter
#include <vector>
int coinChange(std::vector<int>& coins, int amount);
```

```cpp
int coinChange(std::vector<int>& coins, int amount) {
    const int INF = amount + 1;
    std::vector<int> dp(amount + 1, INF);
    dp[0] = 0;
    for (int a = 1; a <= amount; ++a)
        for (int c : coins)
            if (c <= a) dp[a] = std::min(dp[a], dp[a - c] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> c{1,2,5};   if (coinChange(c, 11) != 3)  { std::puts("case1"); return 1; } }
    { vector<int> c{2};       if (coinChange(c, 3)  != -1) { std::puts("case2"); return 1; } }
    { vector<int> c{1};       if (coinChange(c, 0)  != 0)  { std::puts("case3"); return 1; } }
    { vector<int> c{1,2,5};   if (coinChange(c, 100) != 20){ std::puts("case4"); return 1; } }
    { vector<int> c{2,5,10,1};if (coinChange(c, 27) != 4)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```
