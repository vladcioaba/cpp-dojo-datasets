## challenge: Counting Bits
tags: bit-tricks, dynamic-programming
track: faang
difficulty: medium

Given an integer `n`, return an array `ans` of length `n + 1` such that for every `i` in the range `0 <= i <= n`, `ans[i]` is the number of `1` bits in the binary representation of `i`. Aim for a single linear pass.

Constraints: `0 <= n <= 10^5`.

Example: `n = 2` → `[0,1,1]`. Example: `n = 5` → `[0,1,1,2,1,2]`.

hint: Recomputing each popcount independently is O(n log n); you can reuse answers you already have.
hint: The number `i >> 1` is `i` with its lowest bit dropped, so it has the same set bits except possibly that last one.
hint: `ans[i] = ans[i >> 1] + (i & 1)` — take the count for the number half as large, then add 1 if `i` is odd.

```cpp
// starter
#include <vector>
std::vector<int> countBits(int n);
```

```cpp
std::vector<int> countBits(int n) {
    std::vector<int> ans(n + 1, 0);
    for (int i = 1; i <= n; ++i)
        ans[i] = ans[i >> 1] + (i & 1);
    return ans;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { auto r = countBits(2); if (r != vector<int>{0,1,1})           { std::puts("case1"); return 1; } }
    { auto r = countBits(5); if (r != vector<int>{0,1,1,2,1,2})     { std::puts("case2"); return 1; } }
    { auto r = countBits(0); if (r != vector<int>{0})               { std::puts("case3"); return 1; } }
    { auto r = countBits(8); if (r != vector<int>{0,1,1,2,1,2,2,3,1}) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Right-shifting `i` by one discards its least significant bit, giving a strictly smaller index whose popcount is already known. The dropped bit contributes `i & 1` (1 when `i` is odd, 0 when even). So `ans[i] = ans[i >> 1] + (i & 1)` builds each answer in O(1) from a previously computed one, yielding an O(n) time, O(n) space dynamic program.
