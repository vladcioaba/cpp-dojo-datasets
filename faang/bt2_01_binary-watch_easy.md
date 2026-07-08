## challenge: Binary Watch
tags: backtracking, bit-manipulation
track: faang
difficulty: easy

A binary watch has 4 LEDs on the top row representing the hours (`0`–`11`) and 6 LEDs on the bottom row representing the minutes (`0`–`59`). Each LED stands for a power of two, and a row displays the sum of its lit LEDs. Given an integer `turnedOn` (the total number of LEDs that are lit across both rows), return all times the watch could show, in any order. The hour is written without a leading zero; the minute is always written as two digits.

Constraints: `0 <= turnedOn <= 10`. Return each time as a string `"H:MM"` where the hour is `0`–`11` and the minute is `00`–`59`.

Example: `turnedOn = 1` -> `["0:01","0:02","0:04","0:08","0:16","0:32","1:00","2:00","4:00","8:00"]`. Example: `turnedOn = 0` -> `["0:00"]`. Example: `turnedOn = 9` -> `[]` (no valid time can light nine LEDs).

hint: There are only 12 hour values and 60 minute values — 720 combinations in total, which is tiny.
hint: For a fixed hour and minute the number of lit LEDs is `popcount(hour) + popcount(minute)`; keep the pairs whose total equals `turnedOn`.
hint: Format the minute with a leading zero when it is below 10.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> readBinaryWatch(int turnedOn);
```

```cpp
std::vector<std::string> readBinaryWatch(int turnedOn) {
    std::vector<std::string> res;
    for (int h = 0; h < 12; ++h) {
        for (int m = 0; m < 60; ++m) {
            if (__builtin_popcount(h) + __builtin_popcount(m) == turnedOn) {
                std::string t = std::to_string(h) + ":";
                if (m < 10) t += "0";
                t += std::to_string(m);
                res.push_back(t);
            }
        }
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <algorithm>
using std::vector;
using std::string;
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    { auto want = canon({"0:00"});
      if (canon(readBinaryWatch(0)) != want) { std::puts("case0"); return 1; } }
    { auto want = canon({"0:01","0:02","0:04","0:08","0:16","0:32","1:00","2:00","4:00","8:00"});
      if (canon(readBinaryWatch(1)) != want) { std::puts("case1"); return 1; } }
    { auto want = canon({"7:31","7:47","7:55","7:59","11:31","11:47","11:55","11:59"});
      if (canon(readBinaryWatch(8)) != want) { std::puts("case8"); return 1; } }
    { if (!readBinaryWatch(9).empty())  { std::puts("case9");  return 1; } }
    { if (!readBinaryWatch(10).empty()) { std::puts("case10"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The search space is so small that enumerating every hour/minute pair and keeping those whose combined bit count matches `turnedOn` is optimal; the equivalent backtracking view assigns each of the 10 LED bits on or off. Formatting pads the minute to two digits. Work is bounded by the fixed 720 pairs, so effectively O(1).
