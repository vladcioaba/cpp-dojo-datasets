## challenge: Find Median from Data Stream
tags: heap, priority-queue, design, two-heaps
track: faang
difficulty: hard

Design a data structure that supports adding integers from a stream and querying the median of all values seen so far. Implement `MedianFinder` with `addNum(int)` and `findMedian()` (returning a `double`). With an even count the median is the average of the two middle values.

Constraints: `-10^5 <= num <= 10^5`, at most `5*10^4` calls, `findMedian` is only called after at least one `addNum`.

Example: `addNum(1); addNum(2); findMedian()` → `1.5`; then `addNum(3); findMedian()` → `2.0`.

hint: Split the values into a lower half and an upper half; the median lives at the boundary between them.
hint: Keep the lower half in a max-heap and the upper half in a min-heap, balanced so their sizes differ by at most one.
hint: On insert, push then rebalance; the median is the larger heap's top, or the average of both tops when sizes are equal.

```cpp
// starter
#include <queue>
#include <vector>
class MedianFinder {
public:
    MedianFinder();
    void addNum(int num);
    double findMedian();
};
```

```cpp
class MedianFinder {
    std::priority_queue<int> lo;                                        // max-heap, lower half
    std::priority_queue<int, std::vector<int>, std::greater<int>> hi;   // min-heap, upper half
public:
    MedianFinder() {}
    void addNum(int num) {
        lo.push(num);
        hi.push(lo.top()); lo.pop();
        if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }
    double findMedian() {
        if (lo.size() > hi.size()) return lo.top();
        return (lo.top() + hi.top()) / 2.0;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <queue>
#include <vector>
#include <functional>
#include <cmath>
//__USER__
static bool eq(double a, double b) { return std::fabs(a - b) < 1e-6; }
int main() {
    { MedianFinder mf; mf.addNum(1); mf.addNum(2);
      if (!eq(mf.findMedian(), 1.5)) { std::puts("case1"); return 1; }
      mf.addNum(3);
      if (!eq(mf.findMedian(), 2.0)) { std::puts("case2"); return 1; } }
    { MedianFinder mf; mf.addNum(5);
      if (!eq(mf.findMedian(), 5.0)) { std::puts("case3"); return 1; } }
    { MedianFinder mf; mf.addNum(-1); mf.addNum(-2); mf.addNum(-3);
      if (!eq(mf.findMedian(), -2.0)) { std::puts("case4"); return 1; }
      mf.addNum(-4);
      if (!eq(mf.findMedian(), -2.5)) { std::puts("case5"); return 1; } }
    { MedianFinder mf; for (int x : {6,10,2,6,5,0}) mf.addNum(x);
      if (!eq(mf.findMedian(), 5.5)) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maintain two heaps: a max-heap `lo` for the smaller half and a min-heap `hi` for the larger half, keeping `|lo| - |hi| <= 1`. Each `addNum` pushes into `lo`, moves its max into `hi`, then pulls back if `hi` grew larger, so both halves stay sorted at their boundary. The median is `lo`'s top when the count is odd, or the average of both tops when even. O(log n) per insertion, O(1) per query, O(n) space.
