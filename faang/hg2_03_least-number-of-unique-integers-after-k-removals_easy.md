## challenge: Least Number of Unique Integers after K Removals
tags: heap, priority-queue, greedy, hash-table

track: faang
difficulty: easy

Given an array `arr` and an integer `k`, remove exactly `k` elements so that the number of remaining distinct integers is as small as possible. Return that minimum count of remaining unique integers.

Constraints: `1 <= arr.length <= 10^5`, `1 <= arr[i] <= 10^9`, `0 <= k <= arr.length`.

Example: `arr = [5,5,4], k = 1` → `1` (remove the single `4`, leaving only `5`). Example: `arr = [4,3,1,1,3,3,2], k = 3` → `2`.

hint: To eliminate a distinct value entirely you must delete all of its occurrences, so cheaper-to-remove values are the ones with the smallest frequency.
hint: Count frequencies, then spend your `k` removals on the least frequent values first.
hint: Push all frequencies into a min-heap and pop while `k` covers the top frequency.

```cpp
// starter
#include <vector>
int findLeastNumOfUniqueInts(std::vector<int>& arr, int k);
```

```cpp
int findLeastNumOfUniqueInts(std::vector<int>& arr, int k) {
    std::unordered_map<int, int> cnt;
    for (int x : arr) cnt[x]++;
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
    for (auto& [v, c] : cnt) pq.push(c);
    while (k > 0 && !pq.empty()) {
        int f = pq.top();
        if (k >= f) { k -= f; pq.pop(); }
        else break;
    }
    return (int)pq.size();
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <functional>
#include <unordered_map>
using std::vector;
//__USER__
int main() {
    { vector<int> a{5,5,4};           if (findLeastNumOfUniqueInts(a, 1) != 1) { std::puts("case1"); return 1; } }
    { vector<int> a{4,3,1,1,3,3,2};   if (findLeastNumOfUniqueInts(a, 3) != 2) { std::puts("case2"); return 1; } }
    { vector<int> a{1};               if (findLeastNumOfUniqueInts(a, 0) != 1) { std::puts("case3"); return 1; } }
    { vector<int> a{2,1,1,3,3,3};     if (findLeastNumOfUniqueInts(a, 3) != 1) { std::puts("case4"); return 1; } }
    { vector<int> a{1,2,3,4,5};       if (findLeastNumOfUniqueInts(a, 5) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Removing a distinct value costs one deletion per occurrence, so to minimize the surviving distinct count you should knock out the rarest values first. Tally frequencies, load them into a min-heap, and repeatedly pop the smallest as long as `k` can absorb its whole count. When the top frequency exceeds the remaining budget you stop; the heap size is the answer. Counting is O(n) and the heap work is O(u log u) for `u` distinct values, with O(u) space.
