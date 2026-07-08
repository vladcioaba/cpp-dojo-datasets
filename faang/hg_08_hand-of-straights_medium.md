## challenge: Hand of Straights
tags: greedy, hash-table, sorting, ordered-map
track: faang
difficulty: medium

Given an integer array `hand` of card values and an integer `groupSize`, determine whether the cards can be rearranged into groups of exactly `groupSize` consecutive values. Return `true` if it is possible, otherwise `false`.

Constraints: `1 <= hand.length <= 10^4`, `0 <= hand[i] <= 10^9`, `1 <= groupSize <= hand.length`.

Example: `hand = [1,2,3,6,2,3,4,7,8], groupSize = 3` → `true` (`[1,2,3],[2,3,4],[6,7,8]`). Example: `hand = [1,2,3,4,5], groupSize = 4` → `false`.

hint: The total number of cards must be divisible by `groupSize`, or grouping is hopeless.
hint: The smallest remaining card must start a group, forcing the next `groupSize - 1` consecutive values to be present.
hint: An ordered count map lets you always find the current minimum and decrement the run it anchors.

```cpp
// starter
#include <vector>
bool isNStraightHand(std::vector<int>& hand, int groupSize);
```

```cpp
bool isNStraightHand(std::vector<int>& hand, int groupSize) {
    if ((int)hand.size() % groupSize != 0) return false;
    std::map<int, int> cnt;
    for (int x : hand) ++cnt[x];
    while (!cnt.empty()) {
        int start = cnt.begin()->first;
        for (int v = start; v < start + groupSize; ++v) {
            auto it = cnt.find(v);
            if (it == cnt.end()) return false;
            if (--it->second == 0) cnt.erase(it);
        }
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <map>
using std::vector;
//__USER__
int main() {
    { vector<int> h{1,2,3,6,2,3,4,7,8}; if (!isNStraightHand(h, 3)) { std::puts("case1"); return 1; } }
    { vector<int> h{1,2,3,4,5};         if ( isNStraightHand(h, 4)) { std::puts("case2"); return 1; } }
    { vector<int> h{1,1,2,2,3,3};       if (!isNStraightHand(h, 3)) { std::puts("case3"); return 1; } }
    { vector<int> h{1,2,3,4,5,6};       if (!isNStraightHand(h, 2)) { std::puts("case4"); return 1; } }
    { vector<int> h{8,10,12};           if ( isNStraightHand(h, 3)) { std::puts("case5"); return 1; } }
    { vector<int> h{2};                 if (!isNStraightHand(h, 1)) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Grouping requires `hand.length` to be a multiple of `groupSize`. The lowest remaining value has no smaller neighbour to join, so it must open a group and pull the next `groupSize - 1` consecutive values with it; if any is missing the hand fails. A `std::map` gives the running minimum in O(log n) and consuming counts drives the greedy forward. O(n log n) time, O(n) space.
