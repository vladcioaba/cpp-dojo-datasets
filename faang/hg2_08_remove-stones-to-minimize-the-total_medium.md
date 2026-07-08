## challenge: Remove Stones to Minimize the Total
tags: heap, priority-queue, greedy

track: faang
difficulty: medium

You are given an array `piles` where `piles[i]` is the number of stones in the `i`-th pile, and an integer `k`. In one operation you choose any pile and remove `floor(piles[i] / 2)` stones from it (so a pile of size `p` becomes `ceil(p / 2)`). Apply exactly `k` operations and return the minimum possible total number of stones remaining.

Constraints: `1 <= piles.length <= 10^5`, `1 <= piles[i] <= 10^4`, `1 <= k <= 10^5`.

Example: `piles = [5,4,9], k = 2` → `12` (`9→5` then `5→3`, leaving `3+4+5`). Example: `piles = [4,3,6,7], k = 3` → `12`.

hint: Each operation removes `floor(p/2)` stones, so operating on the currently largest pile removes the most.
hint: Always apply the next operation to the biggest pile — a max-heap gives it in O(log n).
hint: Pop the top, replace it with `ceil(p/2)`, push it back, and repeat `k` times while tracking the running total.

```cpp
// starter
#include <vector>
int minStoneSum(std::vector<int>& piles, int k);
```

```cpp
int minStoneSum(std::vector<int>& piles, int k) {
    std::priority_queue<int> pq(piles.begin(), piles.end());
    long long sum = 0;
    for (int p : piles) sum += p;
    for (int i = 0; i < k && !pq.empty(); ++i) {
        int top = pq.top(); pq.pop();
        int removed = top / 2;
        sum -= removed;
        pq.push(top - removed);
    }
    return (int)sum;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
using std::vector;
//__USER__
int main() {
    { vector<int> p{5,4,9};   if (minStoneSum(p, 2) != 12)   { std::puts("case1"); return 1; } }
    { vector<int> p{4,3,6,7}; if (minStoneSum(p, 3) != 12)   { std::puts("case2"); return 1; } }
    { vector<int> p{1};       if (minStoneSum(p, 1) != 1)    { std::puts("case3"); return 1; } }
    { vector<int> p{10000};   if (minStoneSum(p, 1) != 5000) { std::puts("case4"); return 1; } }
    { vector<int> p{100,100}; if (minStoneSum(p, 1) != 150)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Removing `floor(p/2)` from a pile of size `p` deletes more stones the larger `p` is, so every operation should target the current maximum. A max-heap yields that pile in O(log n); replace it with `ceil(p/2)`, adjust the running sum by the removed amount, and push it back. After `k` such greedy steps the heap's contents sum to the minimum achievable total. Building the heap is O(n) and each operation is O(log n), for O(n + k log n) time and O(n) space.
