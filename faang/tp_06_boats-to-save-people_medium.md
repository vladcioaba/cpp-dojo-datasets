## challenge: Boats to Save People
tags: two-pointers, greedy, sorting, array
track: faang
difficulty: medium

You are given an array `people` where `people[i]` is the weight of the i-th person, and an infinite number of boats each with weight capacity `limit`. Each boat carries at most two people at the same time, provided the sum of their weights does not exceed `limit`. Return the minimum number of boats needed to carry every person.

Constraints: `1 <= people.length <= 5 * 10^4`, `1 <= people[i] <= limit <= 3 * 10^4`.

Example: `people = [1,2], limit = 3` → `1`. Example: `people = [3,2,2,1], limit = 3` → `3`. Example: `people = [3,5,3,4], limit = 5` → `4`.

hint: To use the fewest boats, try to pair the heaviest remaining person with someone rather than sending them alone.
hint: Sort the weights, then run two pointers from the lightest and heaviest ends.
hint: The heaviest person always boards; if the lightest also fits alongside them, pair them (advance the light pointer); either way retreat the heavy pointer and count one boat.

```cpp
// starter
#include <vector>
int numRescueBoats(std::vector<int>& people, int limit);
```

```cpp
int numRescueBoats(std::vector<int>& people, int limit) {
    std::sort(people.begin(), people.end());
    int lo = 0, hi = (int)people.size() - 1, boats = 0;
    while (lo <= hi) {
        if (people[lo] + people[hi] <= limit) ++lo;
        --hi;
        ++boats;
    }
    return boats;
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
    { vector<int> p{1,2};       if (numRescueBoats(p, 3) != 1) { std::puts("case1"); return 1; } }
    { vector<int> p{3,2,2,1};   if (numRescueBoats(p, 3) != 3) { std::puts("case2"); return 1; } }
    { vector<int> p{3,5,3,4};   if (numRescueBoats(p, 5) != 4) { std::puts("case3"); return 1; } }
    { vector<int> p{1,1,1,1};   if (numRescueBoats(p, 2) != 2) { std::puts("case4"); return 1; } }
    { vector<int> p{5};         if (numRescueBoats(p, 5) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort the weights and greedily pair from both ends. The heaviest remaining person must occupy a boat; the best companion available is the lightest remaining person, because if even they cannot fit, nobody can. If the pair fits under `limit`, both board and both pointers move inward; otherwise the heavy person rides alone. Each boat count is decided in O(1), so the total is O(n log n) time dominated by the sort, with O(1) extra space.
