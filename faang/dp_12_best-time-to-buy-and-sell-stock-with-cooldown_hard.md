## challenge: Best Time to Buy and Sell Stock with Cooldown
tags: dynamic-programming, state-machine
track: faang
difficulty: hard

Given an array `prices` where `prices[i]` is the price of a stock on day `i`, find the maximum profit from any number of buy/sell transactions, subject to two rules: you may hold at most one share at a time, and after you sell you must wait one full day (a cooldown) before buying again. Return the maximum achievable profit.

Constraints: `1 <= prices.length <= 5000`, `0 <= prices[i] <= 1000`.

Example: `prices = [1,2,3,0,2]` → `3` (buy, sell, cooldown, buy, sell). Example: `prices = [1]` → `0`.

hint: Each day you are in one of three situations: holding a share, having just sold today, or resting and free to buy tomorrow.
hint: Track the best profit for each of those states and update them from the previous day's states.
hint: `sold = hold + price`, `hold = max(hold, rest - price)`, `rest = max(rest, previous sold)`; the answer is `max(sold, rest)` on the last day.

```cpp
// starter
#include <vector>
int maxProfit(std::vector<int>& prices);
```

```cpp
int maxProfit(std::vector<int>& prices) {
    int hold = -1000000000, sold = 0, rest = 0;
    for (int p : prices) {
        int prevSold = sold;
        sold = hold + p;
        hold = std::max(hold, rest - p);
        rest = std::max(rest, prevSold);
    }
    return std::max(sold, rest);
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
    { vector<int> p{1,2,3,0,2};      if (maxProfit(p) != 3) { std::puts("case1"); return 1; } }
    { vector<int> p{1};              if (maxProfit(p) != 0) { std::puts("case2"); return 1; } }
    { vector<int> p{2,1};            if (maxProfit(p) != 0) { std::puts("case3"); return 1; } }
    { vector<int> p{6,1,3,2,4,7};    if (maxProfit(p) != 6) { std::puts("case4"); return 1; } }
    { vector<int> p{1,2,4};          if (maxProfit(p) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Model each day with three running states: hold (currently owning a share), sold (sold today, forcing tomorrow's cooldown), and rest (idle and free to buy). Transitions are sold = hold + price, hold = max(hold, rest - price), and rest = max(rest, previous sold), which bakes the cooldown into the fact that a buy can only follow a rest day. The scan is O(n) time and O(1) space, and the answer is the best of sold or rest at the end.
