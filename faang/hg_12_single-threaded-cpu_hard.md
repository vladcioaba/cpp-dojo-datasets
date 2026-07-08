## challenge: Single-Threaded CPU
tags: heap, priority-queue, sorting, simulation
track: faang
difficulty: hard

You are given `tasks` where `tasks[i] = [enqueueTime, processingTime]`. A single-threaded CPU processes tasks under these rules: it stays idle while no task is available; when free it picks the available task with the smallest `processingTime`, breaking ties by smallest index; once started a task runs to completion uninterrupted. Return the order in which the CPU processes the task indices.

Constraints: `1 <= tasks.length <= 10^5`, `1 <= enqueueTime[i], processingTime[i] <= 10^9`.

Example: `tasks = [[1,2],[2,4],[3,2],[4,1]]` → `[0,2,3,1]`. Example: `tasks = [[7,10],[7,12],[7,5],[7,4],[7,2]]` → `[4,3,2,0,1]`.

hint: Consider tasks in order of when they become available, so sort indices by `enqueueTime`.
hint: Among tasks already available, the CPU's choice depends on `(processingTime, index)` — a min-heap keyed on that pair serves the next one.
hint: Advance a virtual clock: if nothing is available, jump it forward to the next enqueue time; otherwise finish the heap's top and add its processing time.

```cpp
// starter
#include <vector>
std::vector<int> getOrder(std::vector<std::vector<int>>& tasks);
```

```cpp
std::vector<int> getOrder(std::vector<std::vector<int>>& tasks) {
    int n = (int)tasks.size();
    std::vector<int> idx(n);
    for (int i = 0; i < n; ++i) idx[i] = i;
    std::sort(idx.begin(), idx.end(),
              [&](int a, int b){ return tasks[a][0] < tasks[b][0]; });
    std::priority_queue<std::pair<int, int>,
                        std::vector<std::pair<int, int>>,
                        std::greater<std::pair<int, int>>> pq;   // (processingTime, index)
    std::vector<int> order;
    order.reserve(n);
    long long time = 0;
    int i = 0;
    while ((int)order.size() < n) {
        while (i < n && tasks[idx[i]][0] <= time) { pq.push({tasks[idx[i]][1], idx[i]}); ++i; }
        if (pq.empty()) { time = tasks[idx[i]][0]; continue; }
        auto [pt, id] = pq.top(); pq.pop();
        time += pt;
        order.push_back(id);
    }
    return order;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <utility>
#include <functional>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> t{{1,2},{2,4},{3,2},{4,1}}; auto r = getOrder(t);
      if (r != vector<int>({0,2,3,1})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> t{{7,10},{7,12},{7,5},{7,4},{7,2}}; auto r = getOrder(t);
      if (r != vector<int>({4,3,2,0,1})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> t{{1,2}}; auto r = getOrder(t);
      if (r != vector<int>({0})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> t{{5,5},{1,1}}; auto r = getOrder(t);
      if (r != vector<int>({1,0})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> t{{1,7},{2,1},{3,1},{4,1}}; auto r = getOrder(t);
      if (r != vector<int>({0,1,2,3})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort task indices by enqueue time so tasks can be admitted as the clock passes them. A min-heap keyed on `(processingTime, index)` yields exactly the CPU's tie-broken choice among available tasks. Simulate a clock: when the heap is empty jump it to the next enqueue time; otherwise pop the best task, advance the clock by its processing time, and record it. Each task is pushed and popped once. O(n log n) time, O(n) space.
