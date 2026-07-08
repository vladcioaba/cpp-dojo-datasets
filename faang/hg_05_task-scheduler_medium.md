## challenge: Task Scheduler
tags: greedy, heap, array, math
track: faang
difficulty: medium

Given a list of CPU `tasks` (labelled `A`–`Z`) and an integer `n`, each task takes one interval to run and two runs of the *same* task must be separated by at least `n` idle-or-other intervals. Return the minimum number of intervals the CPU needs to finish all tasks.

Constraints: `1 <= tasks.length <= 10^4`, `tasks[i]` is an uppercase English letter, `0 <= n <= 100`.

Example: `tasks = ['A','A','A','B','B','B'], n = 2` → `8` (e.g. `A B _ A B _ A B`). Example: `tasks = ['A','A','A','B','B','B'], n = 0` → `6`.

hint: The most frequent task dictates the skeleton: it forms `fmax - 1` fully-spaced blocks of width `n + 1`, plus a final slot.
hint: Every task tied for the maximum frequency adds one extra slot at the tail.
hint: Idle gaps can be swallowed when there are enough distinct tasks, so the answer is never less than `tasks.length` itself.

```cpp
// starter
#include <vector>
int leastInterval(std::vector<char>& tasks, int n);
```

```cpp
int leastInterval(std::vector<char>& tasks, int n) {
    int cnt[26] = {0};
    for (char c : tasks) ++cnt[c - 'A'];
    int fmax = 0;
    for (int i = 0; i < 26; ++i) fmax = std::max(fmax, cnt[i]);
    int nmax = 0;
    for (int i = 0; i < 26; ++i) if (cnt[i] == fmax) ++nmax;
    long long slots = (long long)(fmax - 1) * (n + 1) + nmax;
    return (int)std::max((long long)tasks.size(), slots);
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
    { vector<char> t{'A','A','A','B','B','B'}; if (leastInterval(t, 2) != 8) { std::puts("case1"); return 1; } }
    { vector<char> t{'A','A','A','B','B','B'}; if (leastInterval(t, 0) != 6) { std::puts("case2"); return 1; } }
    { vector<char> t{'A','A','A','A','A','A','B','C','D','E','F','G'};
      if (leastInterval(t, 2) != 16) { std::puts("case3"); return 1; } }
    { vector<char> t{'A'};                     if (leastInterval(t, 100) != 1) { std::puts("case4"); return 1; } }
    { vector<char> t{'A','A','A','B','B','B','C','C','C','D','D','E','E'};
      if (leastInterval(t, 3) != 13) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Only the maximum frequency `fmax` and how many tasks share it matter. Those `fmax` runs create `fmax - 1` blocks of length `n + 1` (task plus cooldown) followed by a final tail holding every max-frequency task, giving `(fmax - 1) * (n + 1) + nmax`. When there are plenty of distinct tasks the idle slots all get filled, so the answer cannot drop below the total task count; take the larger of the two. O(n) time, O(1) space.
