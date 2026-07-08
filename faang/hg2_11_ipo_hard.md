## challenge: IPO
tags: heap, priority-queue, greedy, sorting

track: faang
difficulty: hard

You will complete at most `k` projects to maximize capital before an IPO. Project `i` yields a one-time pure profit `profits[i]` and requires available capital of at least `capital[i]` to start. Completing a project adds its profit to your capital, so it can fund later projects. Starting with capital `w`, return the maximum capital after finishing at most `k` projects.

Constraints: `1 <= k <= 10^5`, `0 <= w <= 10^9`, `1 <= profits.length == capital.length <= 10^5`, `0 <= profits[i] <= 10^4`, `0 <= capital[i] <= 10^9`.

Example: `k = 2, w = 0, profits = [1,2,3], capital = [0,1,1]` → `4` (do project 0 for `+1` → `w=1`, then project 2 for `+3` → `w=4`). Example: `k = 3, w = 0, profits = [1,2,3], capital = [0,1,2]` → `6`.

hint: At each step you may pick any project you can currently afford; picking the most profitable affordable one is always best because profit is non-negative.
hint: Sort projects by required capital so you can unlock them as your capital grows.
hint: Push every affordable project's profit into a max-heap; take its top, add to `w`, and repeat up to `k` times (stop early if nothing is affordable).

```cpp
// starter
#include <vector>
int findMaximizedCapital(int k, int w, std::vector<int>& profits, std::vector<int>& capital);
```

```cpp
int findMaximizedCapital(int k, int w, std::vector<int>& profits, std::vector<int>& capital) {
    int n = (int)profits.size();
    std::vector<int> idx(n);
    for (int i = 0; i < n; ++i) idx[i] = i;
    std::sort(idx.begin(), idx.end(), [&](int a, int b) { return capital[a] < capital[b]; });
    std::priority_queue<int> pq;  // affordable profits
    int j = 0;
    for (int t = 0; t < k; ++t) {
        while (j < n && capital[idx[j]] <= w) { pq.push(profits[idx[j]]); ++j; }
        if (pq.empty()) break;
        w += pq.top(); pq.pop();
    }
    return w;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> p{1,2,3}, c{0,1,1}; if (findMaximizedCapital(2, 0, p, c) != 4) { std::puts("case1"); return 1; } }
    { vector<int> p{1,2,3}, c{0,1,2}; if (findMaximizedCapital(3, 0, p, c) != 6) { std::puts("case2"); return 1; } }
    { vector<int> p{1,2,3}, c{1,1,2}; if (findMaximizedCapital(1, 2, p, c) != 5) { std::puts("case3"); return 1; } }
    { vector<int> p{1},     c{0};     if (findMaximizedCapital(0, 5, p, c) != 5) { std::puts("case4"); return 1; } }
    { vector<int> p{1,2,3}, c{1,1,2}; if (findMaximizedCapital(2, 0, p, c) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Since profits are non-negative, greedily grabbing the most profitable currently affordable project never hurts. Sort projects by their capital requirement and keep a pointer that, whenever your capital rises, pushes all newly affordable projects' profits into a max-heap. Each of the up-to-`k` rounds unlocks affordable projects, then takes the heap's maximum profit and folds it into `w`; if nothing is affordable you stop. Sorting is O(n log n) and each project enters the heap once, so total time is O((n + k) log n) with O(n) space.
