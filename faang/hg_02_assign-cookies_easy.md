## challenge: Assign Cookies
tags: greedy, sorting, two-pointers
track: faang
difficulty: easy

Each child `i` has a greed factor `g[i]` (the minimum cookie size that satisfies them) and each cookie `j` has size `s[j]`. A cookie can satisfy at most one child, and only if `s[j] >= g[i]`. Return the maximum number of content children.

Constraints: `1 <= g.length <= 3*10^4`, `0 <= s.length <= 3*10^4`, `1 <= g[i], s[j] <= 2^31 - 1`.

Example: `g = [1,2,3], s = [1,1]` → `1` (one cookie of size 1 satisfies the child with greed 1). Example: `g = [1,2], s = [1,2,3]` → `2`.

hint: Sort both arrays; then the smallest cookie that fits the least greedy remaining child is never wasted.
hint: Walk two pointers: advance the cookie pointer always, and the child pointer only when the current cookie satisfies that child.
hint: The child pointer ends up counting exactly how many children were satisfied.

```cpp
// starter
#include <vector>
int findContentChildren(std::vector<int>& g, std::vector<int>& s);
```

```cpp
int findContentChildren(std::vector<int>& g, std::vector<int>& s) {
    std::sort(g.begin(), g.end());
    std::sort(s.begin(), s.end());
    int i = 0, j = 0;
    while (i < (int)g.size() && j < (int)s.size()) {
        if (s[j] >= g[i]) ++i;
        ++j;
    }
    return i;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> g{1,2,3}, s{1,1};       if (findContentChildren(g, s) != 1) { std::puts("case1"); return 1; } }
    { vector<int> g{1,2}, s{1,2,3};       if (findContentChildren(g, s) != 2) { std::puts("case2"); return 1; } }
    { vector<int> g{10,9,8,7}, s{5,6,7,8};if (findContentChildren(g, s) != 2) { std::puts("case3"); return 1; } }
    { vector<int> g{1}, s{};              if (findContentChildren(g, s) != 0) { std::puts("case4"); return 1; } }
    { vector<int> g{1,2,3}, s{3,2,1};     if (findContentChildren(g, s) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort both greed factors and cookie sizes ascending. Sweep with two pointers, giving the smallest sufficient cookie to the least greedy unsatisfied child; if a cookie is too small for the current child it cannot help anyone later either, so discard it. The number of satisfied children is the answer. O(n log n) time from sorting, O(1) extra space.
