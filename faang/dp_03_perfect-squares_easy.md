## challenge: Perfect Squares
tags: dynamic-programming, math
track: faang
difficulty: easy

Given a positive integer `n`, return the least number of perfect-square numbers (`1, 4, 9, 16, ...`) that sum to exactly `n`. A perfect square may be reused any number of times.

Constraints: `1 <= n <= 10^4`.

Example: `n = 12` → `3` (`4 + 4 + 4`). Example: `n = 13` → `2` (`4 + 9`).

hint: This is an unbounded-coin problem where the "coins" are the perfect squares no larger than `n`.
hint: Let `dp[i]` be the fewest squares summing to `i`, built up from `dp[0] = 0`.
hint: `dp[i] = min over squares j*j <= i of dp[i - j*j] + 1`.

```cpp
// starter
int numSquares(int n);
```

```cpp
int numSquares(int n) {
    std::vector<int> dp(n + 1, INT_MAX);
    dp[0] = 0;
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j * j <= i; ++j)
            dp[i] = std::min(dp[i], dp[i - j * j] + 1);
    return dp[n];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <climits>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    if (numSquares(12) != 3)  { std::puts("case1"); return 1; }
    if (numSquares(13) != 2)  { std::puts("case2"); return 1; }
    if (numSquares(1)  != 1)  { std::puts("case3"); return 1; }
    if (numSquares(100)!= 1)  { std::puts("case4"); return 1; }
    if (numSquares(7)  != 4)  { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Treat the perfect squares up to n as an unbounded coin set and minimize the count. dp[i] is the fewest squares summing to i, with dp[i] = min over j*j <= i of dp[i - j*j] + 1 and dp[0] = 0. The double loop runs in O(n * sqrt(n)) time using O(n) space.
