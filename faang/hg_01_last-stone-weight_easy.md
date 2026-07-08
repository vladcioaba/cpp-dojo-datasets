## challenge: Last Stone Weight
tags: heap, priority-queue, greedy
track: faang
difficulty: easy

You are given an array of `stones` weights. On each turn take the two heaviest stones `x <= y` and smash them: if `x == y` both are destroyed, otherwise a new stone of weight `y - x` is created. Repeat until at most one stone remains; return its weight, or `0` if none remain.

Constraints: `1 <= stones.length <= 30`, `1 <= stones[i] <= 1000`.

Example: `stones = [2,7,4,1,8,1]` → `1` (smash 8,7→1; 4,4→0; 2,1,1,1→ … left with 1). Example: `stones = [1]` → `1`.

hint: Each turn you need the two current maxima and you keep inserting new values — a max-heap keeps the largest reachable in O(log n).
hint: Pop the two largest; if they differ, push their difference back into the heap.
hint: Stop when the heap has one element (return it) or is empty (return 0).

```cpp
// starter
#include <vector>
int lastStoneWeight(std::vector<int>& stones);
```

```cpp
int lastStoneWeight(std::vector<int>& stones) {
    std::priority_queue<int> pq(stones.begin(), stones.end());
    while (pq.size() > 1) {
        int y = pq.top(); pq.pop();
        int x = pq.top(); pq.pop();
        if (y != x) pq.push(y - x);
    }
    return pq.empty() ? 0 : pq.top();
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
    { vector<int> s{2,7,4,1,8,1}; if (lastStoneWeight(s) != 1) { std::puts("case1"); return 1; } }
    { vector<int> s{1};           if (lastStoneWeight(s) != 1) { std::puts("case2"); return 1; } }
    { vector<int> s{1,1};         if (lastStoneWeight(s) != 0) { std::puts("case3"); return 1; } }
    { vector<int> s{3,7,2};       if (lastStoneWeight(s) != 2) { std::puts("case4"); return 1; } }
    { vector<int> s{10,4,2,10};   if (lastStoneWeight(s) != 2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A max-heap gives O(log n) access to the two heaviest stones as the multiset keeps changing. Pop the top two; if they are unequal, push back their difference. After every turn the heap shrinks by at least one, so the process ends with zero or one stone. O(n log n) time, O(n) space.
