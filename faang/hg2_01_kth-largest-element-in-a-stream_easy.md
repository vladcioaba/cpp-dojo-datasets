## challenge: Kth Largest Element in a Stream
tags: heap, priority-queue, design, stream
track: faang
difficulty: easy

Design a class `KthLargest` that tracks the k-th largest value in a stream of integers (the k-th largest in sorted order, not the k-th distinct). The constructor receives `k` and an initial array `nums`; each call to `add(val)` appends `val` to the stream and returns the current k-th largest element. It is guaranteed there are always at least `k` elements once `add` is called.

Constraints: `1 <= k <= 10^4`, `0 <= nums.length <= 10^4`, `-10^4 <= nums[i], val <= 10^4`, at most `10^4` calls to `add`.

Example: `KthLargest(3, [4,5,8,2])`; `add(3)` → `4`; `add(5)` → `5`; `add(10)` → `5`; `add(9)` → `8`; `add(4)` → `8`.

hint: You never need the whole sorted stream — only the k largest values matter, and the smallest of those is the answer.
hint: Keep a min-heap capped at size `k`; its top is the k-th largest seen so far.
hint: On each insert push the value, then pop while the heap exceeds size `k`; return the top.

```cpp
// starter
#include <vector>
class KthLargest {
public:
    KthLargest(int k, std::vector<int>& nums);
    int add(int val);
};
```

```cpp
class KthLargest {
    int k;
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
public:
    KthLargest(int k_, std::vector<int>& nums) : k(k_) {
        for (int x : nums) add(x);
    }
    int add(int val) {
        pq.push(val);
        if ((int)pq.size() > k) pq.pop();
        return pq.top();
    }
};
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
    { vector<int> n{4,5,8,2}; KthLargest kl(3, n);
      if (kl.add(3) != 4)  { std::puts("case1"); return 1; }
      if (kl.add(5) != 5)  { std::puts("case2"); return 1; }
      if (kl.add(10) != 5) { std::puts("case3"); return 1; }
      if (kl.add(9) != 8)  { std::puts("case4"); return 1; }
      if (kl.add(4) != 8)  { std::puts("case5"); return 1; } }
    { vector<int> n{}; KthLargest kl(1, n);
      if (kl.add(-3) != -3) { std::puts("case6"); return 1; }
      if (kl.add(-2) != -2) { std::puts("case7"); return 1; }
      if (kl.add(-4) != -2) { std::puts("case8"); return 1; }
      if (kl.add(0) != 0)   { std::puts("case9"); return 1; }
      if (kl.add(4) != 4)   { std::puts("case10"); return 1; } }
    { vector<int> n{0}; KthLargest kl(2, n);
      if (kl.add(-1) != -1) { std::puts("case11"); return 1; }
      if (kl.add(1) != 0)   { std::puts("case12"); return 1; }
      if (kl.add(-2) != 0)  { std::puts("case13"); return 1; }
      if (kl.add(3) != 1)   { std::puts("case14"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The k-th largest element is the minimum of the k largest values seen so far, so a min-heap of capacity `k` is all you need. Each `add` pushes the new value and, if the heap now holds `k+1` elements, pops the smallest; the heap top is then the answer. Every operation is O(log k) and the structure uses O(k) space regardless of how long the stream grows.
