## challenge: Furthest Building You Can Reach
tags: heap, priority-queue, greedy

track: faang
difficulty: medium

You are given an integer array `heights` of building heights, plus `bricks` bricks and `ladders`. Starting at building `0` you move to adjacent buildings left to right. Going to a shorter or equal building is free. Going up by `h` requires either a ladder (unlimited height) or `h` bricks. Return the index of the furthest building you can reach (0-indexed) using the resources optimally.

Constraints: `1 <= heights.length <= 10^5`, `1 <= heights[i] <= 10^6`, `0 <= bricks <= 10^9`, `0 <= ladders <= heights.length`.

Example: `heights = [4,2,7,6,9,14,12], bricks = 5, ladders = 1` → `4` (ladder on the `2→7` climb, bricks elsewhere; you can't pass building 4). Example: `heights = [4,12,2,7,3,18,20,3,19], bricks = 10, ladders = 2` → `7`.

hint: Ladders are most valuable on the biggest climbs, so you'd like to save them for the largest gaps you encounter.
hint: Tentatively assign a ladder to every climb, but keep only the `ladders` largest; the rest must be paid with bricks.
hint: A min-heap of climbs used-so-far lets you evict the smallest climb (pay it with bricks) whenever the heap exceeds `ladders`; fail when bricks go negative.

```cpp
// starter
#include <vector>
int furthestBuilding(std::vector<int>& heights, int bricks, int ladders);
```

```cpp
int furthestBuilding(std::vector<int>& heights, int bricks, int ladders) {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;  // smallest climbs on top
    for (int i = 0; i + 1 < (int)heights.size(); ++i) {
        int diff = heights[i + 1] - heights[i];
        if (diff <= 0) continue;
        pq.push(diff);
        if ((int)pq.size() > ladders) {
            bricks -= pq.top(); pq.pop();
            if (bricks < 0) return i;
        }
    }
    return (int)heights.size() - 1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <functional>
using std::vector;
//__USER__
int main() {
    { vector<int> h{4,2,7,6,9,14,12};             if (furthestBuilding(h, 5, 1)  != 4) { std::puts("case1"); return 1; } }
    { vector<int> h{4,12,2,7,3,18,20,3,19};       if (furthestBuilding(h, 10, 2) != 7) { std::puts("case2"); return 1; } }
    { vector<int> h{14,3,19,3};                   if (furthestBuilding(h, 17, 0) != 3) { std::puts("case3"); return 1; } }
    { vector<int> h{1,2};                         if (furthestBuilding(h, 0, 0)  != 0) { std::puts("case4"); return 1; } }
    { vector<int> h{7,4,2,1};                     if (furthestBuilding(h, 0, 0)  != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Only upward climbs cost anything, and ladders should cover the largest of them. Walk the buildings pushing each positive climb into a min-heap; once the heap holds more climbs than you have ladders, the smallest one must be paid with bricks — subtract it and pop. The moment bricks fall below zero you cannot cross that gap, so return the current index. Reaching the end means every climb was covered. Each climb touches the heap O(log L) for `L` ladders, giving O(n log L) time and O(L) space.
