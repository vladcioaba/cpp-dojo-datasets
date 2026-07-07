## challenge: Koko Eating Bananas
tags: binary-search, greedy
track: faang
difficulty: medium

Koko has `piles` of bananas and `h` hours before the guards return. She eats at a speed of `k` bananas/hour: each hour she picks a pile and eats `k` from it (if the pile has fewer, she finishes it and stops for that hour). Return the minimum integer `k` such that she can finish all bananas within `h` hours.

Constraints: `1 <= piles.length <= 10^4`, `piles.length <= h <= 10^9`, `1 <= piles[i] <= 10^9`.

Example: `piles = [3,6,7,11], h = 8` → `4`. Example: `piles = [30,11,23,4,20], h = 5` → `30`. Example: `h = 6` → `23`.

hint: A faster eating speed is never worse: if speed k finishes in time, every speed above k does too. That monotonicity is the key.
hint: Binary search on the answer itself — the speed k — between 1 and the largest pile.
hint: For a candidate k, the hours needed are the sum of `ceil(pile / k)`; shrink or grow k based on whether it fits within h.

```cpp
// starter
#include <vector>
int minEatingSpeed(std::vector<int>& piles, int h);
```

```cpp
int minEatingSpeed(std::vector<int>& piles, int h) {
    int lo = 1, hi = *std::max_element(piles.begin(), piles.end());
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        long long hours = 0;
        for (int p : piles) hours += (p + mid - 1) / mid;   // ceil(p / mid)
        if (hours <= h) hi = mid;
        else lo = mid + 1;
    }
    return lo;
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
    { vector<int> p{3,6,7,11};        if (minEatingSpeed(p, 8) != 4)  { std::puts("case1"); return 1; } }
    { vector<int> p{30,11,23,4,20};   if (minEatingSpeed(p, 5) != 30) { std::puts("case2"); return 1; } }
    { vector<int> p{30,11,23,4,20};   if (minEatingSpeed(p, 6) != 23) { std::puts("case3"); return 1; } }
    { vector<int> p{312884470};       if (minEatingSpeed(p, 968709470) != 1) { std::puts("case4"); return 1; } }
    { vector<int> p{1,1,1,1};         if (minEatingSpeed(p, 4) != 1)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Binary search over the space of eating speeds. The predicate "can finish within h hours at speed k" is monotonic in k, so binary search finds the smallest feasible k; each check sums `ceil(pile / k)`. O(n log(maxPile)) time, O(1) space.
