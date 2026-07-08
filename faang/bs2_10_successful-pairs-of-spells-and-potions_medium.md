## challenge: Successful Pairs of Spells and Potions
tags: binary-search, array, sorting
track: faang
difficulty: medium

You are given two arrays `spells` and `potions` of positive integers, where `spells[i]` is the strength of the `i`-th spell and `potions[j]` is the strength of the `j`-th potion, and an integer `success`. A pair `(i, j)` is **successful** if `spells[i] * potions[j] >= success`. Return an array `pairs` where `pairs[i]` is the number of potions that form a successful pair with the `i`-th spell.

Constraints: `1 <= spells.length, potions.length <= 10^5`, `1 <= spells[i], potions[i] <= 10^5`, `1 <= success <= 10^10`.

Example: `spells = [5,1,3], potions = [1,2,3,4,5], success = 7` → `[4,0,3]`. Example: `spells = [3,1,2], potions = [8,5,8], success = 16` → `[2,0,2]`.

hint: Sort `potions` once. Then for a fixed spell, the potions that succeed form a suffix of the sorted array.
hint: For spell strength `s`, you need `potions[j] >= success / s`; binary search for the first such potion and count the rest.
hint: Compare with `(long long)s * potions[mid] >= success` to keep the product exact — `s * potion` can reach `10^10`, beyond 32-bit range.

```cpp
// starter
#include <vector>
std::vector<int> successfulPairs(std::vector<int>& spells, std::vector<int>& potions, long long success);
```

```cpp
std::vector<int> successfulPairs(std::vector<int>& spells, std::vector<int>& potions, long long success) {
    std::sort(potions.begin(), potions.end());
    int m = (int)potions.size();
    std::vector<int> res;
    res.reserve(spells.size());
    for (int s : spells) {
        int lo = 0, hi = m;                       // first index with s * potions[idx] >= success
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if ((long long)s * potions[mid] >= success) hi = mid;
            else lo = mid + 1;
        }
        res.push_back(m - lo);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
static bool eq(const vector<int>& a, const vector<int>& b) {
    if (a.size() != b.size()) return false;
    for (size_t i = 0; i < a.size(); ++i) if (a[i] != b[i]) return false;
    return true;
}
int main() {
    { vector<int> s{5,1,3}, p{1,2,3,4,5}; auto r = successfulPairs(s, p, 7);
      if (!eq(r, {4,0,3})) { std::puts("case1"); return 1; } }
    { vector<int> s{3,1,2}, p{8,5,8};     auto r = successfulPairs(s, p, 16);
      if (!eq(r, {2,0,2})) { std::puts("case2"); return 1; } }
    { vector<int> s{1}, p{1};             auto r = successfulPairs(s, p, 1);
      if (!eq(r, {1})) { std::puts("case3"); return 1; } }
    { vector<int> s{1}, p{1};             auto r = successfulPairs(s, p, 2);
      if (!eq(r, {0})) { std::puts("case4"); return 1; } }
    { vector<int> s{100000}, p{100000};   auto r = successfulPairs(s, p, 10000000000LL);
      if (!eq(r, {1})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort `potions` once in O(P log P). Multiplication is monotone in potion strength, so for a fixed spell the successful potions are exactly the suffix at or beyond the first potion satisfying `spells[i] * potion >= success`. Binary search that boundary per spell and count `m - lo` potions. Do the product in `long long` since it can reach `10^10`. Total O((S + P) log P) time.
