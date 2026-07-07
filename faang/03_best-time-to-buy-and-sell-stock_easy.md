## challenge: Best Time to Buy and Sell Stock
tags: sliding-window, array, dynamic-programming
track: faang
difficulty: easy

You are given an array `prices` where `prices[i]` is the price of a stock on day `i`. Buy on one day and sell on a later day to maximize profit. Return the maximum profit, or `0` if no profit is possible.

Constraints: `1 <= prices.length <= 10^5`, `0 <= prices[i] <= 10^4`.

Example: `prices = [7,1,5,3,6,4]` → `5` (buy at 1, sell at 6). Example: `prices = [7,6,4,3,1]` → `0` (never profitable).

hint: You must buy before you sell, so at each day ask: if I sell today, what is the cheapest I could have bought earlier?
hint: Sweep left to right tracking the minimum price seen so far and the best profit against it.

```cpp
// starter
#include <vector>
int maxProfit(std::vector<int>& prices);
```

```cpp
int maxProfit(std::vector<int>& prices) {
    int best = 0, minSoFar = INT_MAX;
    for (int p : prices) {
        minSoFar = std::min(minSoFar, p);
        best = std::max(best, p - minSoFar);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <climits>
using std::vector;
//__USER__
int main() {
    { vector<int> p{7,1,5,3,6,4}; if (maxProfit(p) != 5) { std::puts("case1"); return 1; } }
    { vector<int> p{7,6,4,3,1};   if (maxProfit(p) != 0) { std::puts("case2"); return 1; } }
    { vector<int> p{1};           if (maxProfit(p) != 0) { std::puts("case3"); return 1; } }
    { vector<int> p{2,4,1};       if (maxProfit(p) != 2) { std::puts("case4"); return 1; } }
    { vector<int> p{3,2,6,5,0,3}; if (maxProfit(p) != 4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A single pass keeps the running minimum price and the best profit `price - minSoFar`. Since the buy day is always at or before the sell day, one scan is enough. O(n) time, O(1) space.
