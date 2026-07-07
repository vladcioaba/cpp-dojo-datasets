## challenge: Daily Temperatures
tags: monotonic-stack, stack, array
track: faang
difficulty: medium

Given a list of daily `temperatures`, return an array `answer` where `answer[i]` is the number of days you must wait after day `i` for a warmer temperature. If no future day is warmer, put `0`.

Constraints: `1 <= temperatures.length <= 10^5`, `30 <= temperatures[i] <= 100`.

Example: `[73,74,75,71,69,72,76,73]` → `[1,1,4,2,1,1,0,0]`. Example: `[30,40,50,60]` → `[1,1,1,0]`. Example: `[30,60,90]` → `[1,1,0]`.

```cpp
// starter
#include <vector>
std::vector<int> dailyTemperatures(std::vector<int>& temperatures);
```

```cpp
std::vector<int> dailyTemperatures(std::vector<int>& temperatures) {
    int n = (int)temperatures.size();
    std::vector<int> res(n, 0), st;   // st holds indices, temps decreasing
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && temperatures[i] > temperatures[st.back()]) {
            res[st.back()] = i - st.back();
            st.pop_back();
        }
        st.push_back(i);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> t{73,74,75,71,69,72,76,73};
      if (dailyTemperatures(t) != vector<int>({1,1,4,2,1,1,0,0})) { std::puts("case1"); return 1; } }
    { vector<int> t{30,40,50,60};
      if (dailyTemperatures(t) != vector<int>({1,1,1,0})) { std::puts("case2"); return 1; } }
    { vector<int> t{30,60,90};
      if (dailyTemperatures(t) != vector<int>({1,1,0})) { std::puts("case3"); return 1; } }
    { vector<int> t{100};
      if (dailyTemperatures(t) != vector<int>({0})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```
