## challenge: Maximum Points You Can Obtain from Cards
tags: array, sliding-window, prefix-sum
track: faang
difficulty: easy

There are several cards arranged in a row, each with an integer score given by `cardPoints`. In one move you take a single card from either the beginning or the end of the row. You must take exactly `k` cards. Return the maximum total score obtainable.

Constraints: `1 <= cardPoints.length <= 10^5`, `1 <= cardPoints[i] <= 10^4`, `1 <= k <= cardPoints.length`.

Example: `cardPoints = [1,2,3,4,5,6,1], k = 3` → `12` (take the last three cards `5,6,1`). Example: `cardPoints = [2,2,2], k = 2` → `4`. Example: `cardPoints = [9,7,7,9,7,7,9], k = 7` → `55` (take every card).

hint: Taking `k` cards from the two ends is the same as leaving a single contiguous block of `n - k` cards untouched in the middle.

hint: The score you keep is fixed at `total - (sum of the untaken middle block)`, so maximizing the score means minimizing that middle block's sum.

hint: Slide a window of width `n - k` to find its minimum sum; if `k == n` the window is empty and you take everything.

```cpp
// starter
#include <vector>
int maxScore(std::vector<int>& cardPoints, int k);
```

```cpp
int maxScore(std::vector<int>& cardPoints, int k) {
    int n = cardPoints.size();
    int total = 0;
    for (int x : cardPoints) total += x;
    int w = n - k;
    if (w == 0) return total;
    int sum = 0;
    for (int i = 0; i < w; ++i) sum += cardPoints[i];
    int minWin = sum;
    for (int i = w; i < n; ++i) {
        sum += cardPoints[i] - cardPoints[i - w];
        minWin = std::min(minWin, sum);
    }
    return total - minWin;
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
    { vector<int> c{1,2,3,4,5,6,1}; if (maxScore(c,3)!=12) { std::puts("case1"); return 1; } }
    { vector<int> c{2,2,2}; if (maxScore(c,2)!=4) { std::puts("case2"); return 1; } }
    { vector<int> c{9,7,7,9,7,7,9}; if (maxScore(c,7)!=55) { std::puts("case3"); return 1; } }
    { vector<int> c{1,1000,1}; if (maxScore(c,1)!=1) { std::puts("case4"); return 1; } }
    { vector<int> c{1,79,80,1,1,1,200,1}; if (maxScore(c,3)!=202) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Instead of choosing which ends to peel, observe that the cards you do not take form one contiguous block of length `n - k` in the middle. The kept score equals the grand total minus that block's sum, so the problem reduces to finding the minimum-sum window of width `n - k`. Compute that window's sum incrementally as you slide it, tracking the smallest, and subtract from the total. The special case `k == n` leaves no block, so the answer is the whole sum. O(n) time, O(1) space.
