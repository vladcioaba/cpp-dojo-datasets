## challenge: Matchsticks to Square
tags: backtracking, bitmask, dynamic-programming
track: faang
difficulty: medium

You are given an integer array `matchsticks` where `matchsticks[i]` is the length of the i-th matchstick. You want to use every matchstick to form a square: you must use all of them and may not break any, though you may link several sticks end to end to make one side. Return `true` if you can form a square, and `false` otherwise.

Constraints: `1 <= matchsticks.length <= 15`, `1 <= matchsticks[i] <= 10^8`.

Example: `matchsticks = [1,1,2,2,2]` -> `true` (two sides of length 2, plus two sides each made of `1 + 1`). Example: `matchsticks = [3,3,3,3,4]` -> `false`. Example: `matchsticks = [1,1,1,1]` -> `true`.

hint: The perimeter must be divisible by 4, and no single stick may exceed the side length `sum / 4`.
hint: Assign each stick to one of the four sides with backtracking, skipping any side that would overflow the target length.
hint: Sorting the sticks in descending order, and not retrying a side whose running length equals an earlier side's, prunes the search dramatically.

```cpp
// starter
#include <vector>
bool makesquare(std::vector<int>& matchsticks);
```

```cpp
bool makesquare(std::vector<int>& matchsticks) {
    long long sum = 0;
    for (int x : matchsticks) sum += x;
    if (matchsticks.empty() || sum % 4 != 0) return false;
    long long side = sum / 4;
    std::sort(matchsticks.rbegin(), matchsticks.rend());
    if (matchsticks[0] > side) return false;
    std::vector<long long> sides(4, 0);
    int n = (int)matchsticks.size();
    std::function<bool(int)> dfs = [&](int i) -> bool {
        if (i == n) return true;
        for (int s = 0; s < 4; ++s) {
            if (sides[s] + matchsticks[i] > side) continue;
            if (s > 0 && sides[s] == sides[s - 1]) continue;
            sides[s] += matchsticks[i];
            if (dfs(i + 1)) return true;
            sides[s] -= matchsticks[i];
        }
        return false;
    };
    return dfs(0);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <functional>
using std::vector;
//__USER__
int main() {
    struct T { vector<int> a; bool want; };
    T tests[] = {
        {{1,1,2,2,2}, true},
        {{3,3,3,3,4}, false},
        {{1,1,1,1}, true},
        {{5,5,5,5,4,4,4,4,3,3,3,3}, true},
        {{3,3,3,3,3}, false},
        {{2,2,2,2,2,2,2,2}, true},
        {{1,2,3,4,5,6,7,8}, true},
        {{1,1,1,1,1,1,1,5}, false}
    };
    for (auto& t : tests) {
        vector<int> a = t.a;
        if (makesquare(a) != t.want) { std::puts("fail"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Reject immediately when the perimeter is not divisible by four or the longest stick exceeds the side length `sum / 4`. Otherwise assign sticks (largest first) to four running buckets with backtracking, never letting a bucket exceed the target and skipping a bucket whose current length duplicates the one before it. Descending order and the duplicate-bucket skip make the exponential search fast in practice; worst case is O(4^n).
