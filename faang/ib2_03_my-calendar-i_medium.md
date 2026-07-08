## challenge: My Calendar I
tags: intervals, design, ordered-set
track: faang
difficulty: medium

Implement a `MyCalendar` class to store your events without a double booking. A double booking happens when two events share some nonempty half-open interval — that is, they have a common integer time. An event is a half-open interval `[start, end)` covering all times `t` with `start <= t < end`. The method `book(start, end)` returns `true` and records the event if it can be added without causing a double booking; otherwise it returns `false` and adds nothing.

Constraints: `0 <= start < end <= 10^9`, at most `1000` calls to `book`.

Example: `book(10, 20)` → `true`, `book(15, 25)` → `false` (overlaps `[10,20)`), `book(20, 30)` → `true` (touches at 20 but shares no time).

hint: Two half-open intervals `[s1,e1)` and `[s2,e2)` overlap exactly when `s1 < e2` and `s2 < e1`.
hint: Keep the accepted events; for a new one, reject it if it overlaps any stored event, else store it.
hint: Touching endpoints (`end == other.start`) do not conflict because the interval is half-open.

```cpp
// starter
class MyCalendar {
public:
    MyCalendar();
    bool book(int start, int end);
};
```

```cpp
class MyCalendar {
    std::vector<std::pair<int,int>> events;
public:
    MyCalendar() {}
    bool book(int start, int end) {
        for (auto& e : events)
            if (start < e.second && e.first < end) return false;
        events.push_back({start, end});
        return true;
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
    { MyCalendar c;
      if (c.book(10, 20) != true)  { std::puts("case1"); return 1; }
      if (c.book(15, 25) != false) { std::puts("case2"); return 1; }
      if (c.book(20, 30) != true)  { std::puts("case3"); return 1; }
      if (c.book(5, 10)  != true)  { std::puts("case4"); return 1; }
      if (c.book(8, 12)  != false) { std::puts("case5"); return 1; }
      if (c.book(25, 40) != false) { std::puts("case6"); return 1; } }
    { MyCalendar c;
      if (c.book(47, 50) != true)  { std::puts("case7"); return 1; }
      if (c.book(33, 41) != true)  { std::puts("case8"); return 1; }
      if (c.book(39, 45) != false) { std::puts("case9"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Store each accepted event as a half-open interval. Two intervals `[s1,e1)` and `[s2,e2)` conflict precisely when `s1 < e2 && s2 < e1`; the strict inequalities let events that merely touch at an endpoint coexist. For each `book`, scan the stored events and reject on the first conflict, otherwise append. With at most 1000 calls the linear scan gives O(n) per booking and O(n^2) overall, well within limits; a balanced BST of intervals would reduce a booking to O(log n).
