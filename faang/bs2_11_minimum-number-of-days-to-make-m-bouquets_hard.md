## challenge: Minimum Number of Days to Make m Bouquets
tags: binary-search, array
track: faang
difficulty: hard

You are given an array `bloomDay`, an integer `m`, and an integer `k`. Flower `i` blooms on day `bloomDay[i]`. To make one bouquet you need `k` **adjacent** bloomed flowers from the garden. Return the minimum number of days you must wait to make `m` bouquets, or `-1` if it is impossible to make them at all.

Constraints: `1 <= bloomDay.length <= 10^5`, `1 <= bloomDay[i] <= 10^9`, `1 <= m <= 10^6`, `1 <= k <= bloomDay.length`.

Example: `bloomDay = [1,10,3,10,2], m = 3, k = 1` → `3`. Example: `bloomDay = [1,10,3,10,2], m = 3, k = 2` → `-1` (need 6 flowers, only 5 exist). Example: `bloomDay = [7,7,7,7,12,7,7], m = 2, k = 3` → `12`.

hint: You need `m * k` flowers in total; if that exceeds the garden size, it is outright impossible — return `-1`.
hint: "Can I make `m` bouquets by day `d`?" is monotone: waiting longer never makes it harder. Binary search the day.
hint: To test a day `d`, sweep once counting runs of consecutive flowers with `bloomDay[i] <= d`, forming a bouquet every `k` in a row.

```cpp
// starter
#include <vector>
int minDays(std::vector<int>& bloomDay, int m, int k);
```

```cpp
int minDays(std::vector<int>& bloomDay, int m, int k) {
    long long need = (long long)m * k;
    int n = (int)bloomDay.size();
    if (need > n) return -1;
    int lo = bloomDay[0], hi = bloomDay[0];
    for (int b : bloomDay) { if (b < lo) lo = b; if (b > hi) hi = b; }
    auto can = [&](int day) {
        int bouquets = 0, run = 0;
        for (int b : bloomDay) {
            if (b <= day) { if (++run == k) { ++bouquets; run = 0; } }
            else run = 0;
        }
        return bouquets >= m;
    };
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (can(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> b{1,10,3,10,2};      if (minDays(b, 3, 1) != 3)  { std::puts("case1"); return 1; } }
    { vector<int> b{1,10,3,10,2};      if (minDays(b, 3, 2) != -1) { std::puts("case2"); return 1; } }
    { vector<int> b{7,7,7,7,12,7,7};   if (minDays(b, 2, 3) != 12) { std::puts("case3"); return 1; } }
    { vector<int> b{1,10,2,9,3,8,4,7,5,6}; if (minDays(b, 4, 2) != 9) { std::puts("case4"); return 1; } }
    { vector<int> b{1000000000,1000000000}; if (minDays(b, 1, 1) != 1000000000) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** First rule out impossibility: making `m` bouquets of `k` flowers needs `m * k` flowers, so if that exceeds the array length the answer is `-1`. Otherwise binary search the day. The feasibility check "at most `day` yields `>= m` bouquets" is monotone in `day`; evaluate it in O(n) by scanning for runs of already-bloomed flowers and consuming `k` at a time. Search `day` over `[min(bloomDay), max(bloomDay)]`. O(n log(max)) time, O(1) space.
