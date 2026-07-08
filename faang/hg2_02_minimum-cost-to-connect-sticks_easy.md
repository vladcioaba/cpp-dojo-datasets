## challenge: Minimum Cost to Connect Sticks
tags: heap, priority-queue, greedy

track: faang
difficulty: easy

You have some `sticks` with positive integer lengths. You can connect any two sticks of lengths `x` and `y` into one stick of length `x + y`, paying a cost of `x + y`. Keep connecting until a single stick remains and return the minimum total cost.

Constraints: `1 <= sticks.length <= 10^4`, `1 <= sticks[i] <= 10^4`.

Example: `sticks = [2,4,3]` → `14` (connect `2+3=5` costing `5`, then `5+4=9` costing `9`; total `14`). Example: `sticks = [1,8,3,5]` → `30`. Example: `sticks = [5]` → `0`.

hint: The cost of an early merge is paid again in every later merge that includes it, so small sticks should be combined first.
hint: This is Huffman merging: always join the two shortest available sticks.
hint: A min-heap gives the two shortest in O(log n); push their sum back and accumulate the cost.

```cpp
// starter
#include <vector>
int connectSticks(std::vector<int>& sticks);
```

```cpp
int connectSticks(std::vector<int>& sticks) {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq(sticks.begin(), sticks.end());
    long long total = 0;
    while (pq.size() > 1) {
        long long a = pq.top(); pq.pop();
        long long b = pq.top(); pq.pop();
        total += a + b;
        pq.push((int)(a + b));
    }
    return (int)total;
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
    { vector<int> s{2,4,3};   if (connectSticks(s) != 14) { std::puts("case1"); return 1; } }
    { vector<int> s{1,8,3,5}; if (connectSticks(s) != 30) { std::puts("case2"); return 1; } }
    { vector<int> s{5};       if (connectSticks(s) != 0)  { std::puts("case3"); return 1; } }
    { vector<int> s{1,1};     if (connectSticks(s) != 2)  { std::puts("case4"); return 1; } }
    { vector<int> s{3,2,4,1}; if (connectSticks(s) != 19) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because every merge's cost is added into all subsequent merges that consume the resulting stick, the optimal strategy is the Huffman rule: always combine the two currently shortest sticks. A min-heap yields those two in O(log n); push back their sum and add it to the running total. Each of the `n-1` merges touches the heap a constant number of times, so the algorithm is O(n log n) time and O(n) space.
