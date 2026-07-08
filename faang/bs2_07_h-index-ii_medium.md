## challenge: H-Index II
tags: binary-search, array
track: faang
difficulty: medium

Given an array `citations` **sorted in ascending order**, where `citations[i]` is the number of citations a researcher's `i`-th paper received, return the researcher's h-index. The h-index is the maximum value `h` such that the researcher has at least `h` papers with `h` or more citations each. You must solve it in O(log n).

Constraints: `1 <= citations.length <= 10^5`, `0 <= citations[i] <= 1000`, `citations` is sorted in ascending order.

Example: `citations = [0,1,3,5,6]` → `3` (three papers have `>= 3` citations). Example: `citations = [1,2,100]` → `2`. Example: `citations = [0]` → `0`.

hint: Because the array is sorted, the papers from index `i` to the end are the `n - i` most-cited ones.
hint: At index `i`, "the top `n - i` papers each have `>= n - i` citations" holds exactly when `citations[i] >= n - i` — a monotone condition.
hint: Binary search for the first index `i` where `citations[i] >= n - i`; the h-index is then `n - i`.

```cpp
// starter
#include <vector>
int hIndex(std::vector<int>& citations);
```

```cpp
int hIndex(std::vector<int>& citations) {
    int n = (int)citations.size();
    int lo = 0, hi = n;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (citations[mid] >= n - mid) hi = mid;   // this cutoff works, try smaller index
        else lo = mid + 1;
    }
    return n - lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> c{0,1,3,5,6}; if (hIndex(c) != 3) { std::puts("case1"); return 1; } }
    { vector<int> c{1,2,100};   if (hIndex(c) != 2) { std::puts("case2"); return 1; } }
    { vector<int> c{0};         if (hIndex(c) != 0) { std::puts("case3"); return 1; } }
    { vector<int> c{100};       if (hIndex(c) != 1) { std::puts("case4"); return 1; } }
    { vector<int> c{0,0};       if (hIndex(c) != 0) { std::puts("case5"); return 1; } }
    { vector<int> c{4,4,4,4};   if (hIndex(c) != 4) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** With the array sorted ascending, choosing index `i` as the cutoff means the `n - i` papers from `i` onward each have at least `citations[i]` citations. The h-index of `n - i` is achievable exactly when `citations[i] >= n - i`, and this predicate is monotone in `i` (once true, it stays true). Binary search for the smallest such `i`; the answer is `n - i`. O(log n) time, O(1) space.
