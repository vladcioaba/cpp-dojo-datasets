## challenge: Find the Town Judge
tags: graph, in-degree, out-degree
track: faang
difficulty: easy

In a town of `n` people labeled from `1` to `n`, a rumor says one of them is secretly the town judge. If the judge exists then the judge trusts nobody, everybody except the judge trusts the judge, and exactly one person satisfies both properties. You are given an array `trust` where `trust[i] = [a, b]` means person `a` trusts person `b`. Return the label of the town judge if one exists and can be identified, otherwise return `-1`.

Constraints: `1 <= n <= 1000`, `0 <= trust.length <= 10^4`, `trust[i].length == 2`, all `trust[i]` distinct, `1 <= a, b <= n`, `a != b`.

Example: `n = 2, trust = [[1,2]]` → `2`. Example: `n = 3, trust = [[1,3],[2,3]]` → `3`. Example: `n = 3, trust = [[1,3],[2,3],[3,1]]` → `-1` (person 3 trusts someone, so cannot be the judge).

hint: Think of each trust as a directed edge. The judge has a very specific in-degree and out-degree signature.
hint: The judge must be trusted by everyone else (`in-degree == n-1`) while trusting no one (`out-degree == 0`). Track `in - out` per person.

```cpp
// starter
#include <vector>
int findJudge(int n, std::vector<std::vector<int>>& trust);
```

```cpp
int findJudge(int n, std::vector<std::vector<int>>& trust) {
    std::vector<int> score(n + 1, 0);
    for (auto& t : trust) { --score[t[0]]; ++score[t[1]]; }
    for (int i = 1; i <= n; ++i) if (score[i] == n - 1) return i;
    return -1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> t{{1,2}}; if (findJudge(2, t) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> t{{1,3},{2,3}}; if (findJudge(3, t) != 3) { std::puts("case2"); return 1; } }
    { vector<vector<int>> t{{1,3},{2,3},{3,1}}; if (findJudge(3, t) != -1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> t{}; if (findJudge(1, t) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> t{{1,2},{2,3}}; if (findJudge(3, t) != -1) { std::puts("case5"); return 1; } }
    { vector<vector<int>> t{{1,3},{1,4},{2,3},{2,4},{4,3}}; if (findJudge(4, t) != 3) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Model trust as directed edges. The judge is trusted by all `n-1` others and trusts nobody, so `in-degree - out-degree == n-1`. Maintain a single `score` array: subtract one for every person who trusts (out-edge) and add one for every person trusted (in-edge). Any person whose score equals `n-1` is the judge. One pass over the edges plus one over the people gives O(n + E) time, O(n) space.
