## challenge: Online Stock Span
tags: monotonic-stack, stack, design
track: faang
difficulty: medium

Design an algorithm that collects daily price quotes for a stock and, for each day, returns the stock's span. The span on a given day is the maximum number of consecutive days (ending with that day and going backward) for which the price was less than or equal to that day's price. Implement the `StockSpanner` class: each call to `next(price)` provides the price for the current day and returns its span.

Constraints: `1 <= price <= 10^5`; at most `10^4` calls are made to `next`.

Example: prices `[100,80,60,70,60,75,85]` yield spans `[1,1,1,2,1,4,6]`.

hint: A day's span skips over earlier days whose price is less than or equal to today's; those days can never limit a later, higher day again.
hint: Keep a stack of past days, but store each day together with the span it already absorbed, so you can collapse runs.
hint: On `next`, start span `1`, then while the stack top's price `<=` the new price, pop it and add its stored span; push the new price with the accumulated span.

```cpp
// starter
class StockSpanner {
public:
    StockSpanner();
    int next(int price);
};
```

```cpp
class StockSpanner {
    std::vector<std::pair<int,int>> st;   // (price, span) with strictly decreasing prices
public:
    StockSpanner() {}
    int next(int price) {
        int span = 1;
        while (!st.empty() && st.back().first <= price) {
            span += st.back().second;
            st.pop_back();
        }
        st.push_back({price, span});
        return span;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <utility>
using std::vector;
//__USER__
int main() {
    { StockSpanner s;
      int prices[] = {100,80,60,70,60,75,85};
      int expect[] = {1,1,1,2,1,4,6};
      for (int i = 0; i < 7; ++i)
          if (s.next(prices[i]) != expect[i]) { std::puts("case1"); return 1; } }
    { StockSpanner s;
      int prices[] = {31,41,48,59,79};
      int expect[] = {1,2,3,4,5};
      for (int i = 0; i < 5; ++i)
          if (s.next(prices[i]) != expect[i]) { std::puts("case2"); return 1; } }
    { StockSpanner s;
      int prices[] = {10,10,10};
      int expect[] = {1,2,3};
      for (int i = 0; i < 3; ++i)
          if (s.next(prices[i]) != expect[i]) { std::puts("case3"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maintain a monotonic stack of `(price, span)` pairs whose prices strictly decrease from bottom to top. For a new price, its span begins at 1; every stack entry with price `<=` the new price is "absorbed": pop it and add its stored span, because those days are dominated by today. Push the new price with the total span it accumulated. Each day is pushed and popped at most once across all calls, so `next` is amortized O(1) and the structure uses O(n) space.
