## challenge: Grumpy Bookstore Owner
tags: array, sliding-window
track: faang
difficulty: medium

A bookstore owner is open for `n` minutes. `customers[i]` customers arrive during minute `i` and all leave at the end of that minute. On minute `i` the owner is grumpy if `grumpy[i] == 1`, and grumpy minutes make that minute's customers unsatisfied; otherwise they leave satisfied. The owner knows a secret technique that keeps them calm for exactly `minutes` consecutive minutes, usable only once. Return the maximum number of customers that can be satisfied throughout the day.

Constraints: `n == customers.length == grumpy.length`, `1 <= minutes <= n <= 2 * 10^4`, `0 <= customers[i] <= 1000`, `grumpy[i]` is `0` or `1`.

Example: `customers = [1,0,1,2,1,1,7,5], grumpy = [0,1,0,1,0,1,0,1], minutes = 3` → `16`. Example: `customers = [1], grumpy = [0], minutes = 1` → `1`.

hint: Customers on non-grumpy minutes are satisfied no matter what — sum them as a fixed baseline.
hint: The technique only ever helps on grumpy minutes, so the extra it recovers is the sum of grumpy-minute customers inside the chosen window.
hint: Slide a fixed window of length `minutes`, tracking the largest recoverable extra, and add it to the baseline.

```cpp
// starter
#include <vector>
int maxSatisfied(std::vector<int>& customers, std::vector<int>& grumpy, int minutes);
```

```cpp
int maxSatisfied(std::vector<int>& customers, std::vector<int>& grumpy, int minutes) {
    int base = 0;
    for (int i = 0; i < (int)customers.size(); ++i)
        if (grumpy[i] == 0) base += customers[i];
    int win = 0, extra = 0;
    for (int i = 0; i < (int)customers.size(); ++i) {
        if (grumpy[i] == 1) win += customers[i];
        if (i >= minutes && grumpy[i - minutes] == 1) win -= customers[i - minutes];
        extra = std::max(extra, win);
    }
    return base + extra;
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
    { vector<int> c{1,0,1,2,1,1,7,5}, g{0,1,0,1,0,1,0,1}; if (maxSatisfied(c,g,3)!=16) { std::puts("case1"); return 1; } }
    { vector<int> c{1}, g{0}; if (maxSatisfied(c,g,1)!=1) { std::puts("case2"); return 1; } }
    { vector<int> c{4,10,10}, g{1,1,0}; if (maxSatisfied(c,g,2)!=24) { std::puts("case3"); return 1; } }
    { vector<int> c{2,6,6,9}, g{0,0,1,1}; if (maxSatisfied(c,g,1)!=17) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Split the total into two parts. The baseline is every customer on a non-grumpy minute — they are satisfied regardless. The technique can only rescue customers on grumpy minutes, so its value is the sum of grumpy-minute customers covered by the `minutes`-length window. Slide that fixed window across the day, keeping a running sum of grumpy-minute customers inside it and remembering the maximum such sum. The answer is the baseline plus that best extra. O(n) time, O(1) space.
