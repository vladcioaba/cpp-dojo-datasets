## challenge: Intersection of Two Arrays II
tags: array, hash-table, counting, arrays-hashing
track: faang
difficulty: medium

Given two integer arrays `nums1` and `nums2`, return an array of their intersection. Each element in the result must appear as many times as it shows in both arrays, and you may return the result in any order.

Constraints: `1 <= nums1.length, nums2.length <= 1000`, `0 <= nums1[i], nums2[i] <= 1000`.

Example: `nums1 = [1,2,2,1], nums2 = [2,2]` → `[2,2]`. Example: `nums1 = [4,9,5], nums2 = [9,4,9,8,4]` → `[4,9]` (also `[9,4]`). Example: `nums1 = [1,1], nums2 = [2,2]` → `[]`.

hint: Because multiplicity matters, plain set intersection is wrong; you must respect how many times each value is shared.
hint: Count the occurrences of each value in one array with a hash map.
hint: Scan the other array; whenever a value still has a positive remaining count, emit it once and decrement the count so it is not over-reported.

```cpp
// starter
#include <vector>
std::vector<int> intersect(std::vector<int>& nums1, std::vector<int>& nums2);
```

```cpp
std::vector<int> intersect(std::vector<int>& nums1, std::vector<int>& nums2) {
    std::unordered_map<int, int> cnt;
    for (int x : nums1) cnt[x]++;
    std::vector<int> res;
    for (int x : nums2) {
        auto it = cnt.find(x);
        if (it != cnt.end() && it->second > 0) {
            res.push_back(x);
            it->second--;
        }
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <algorithm>
using std::vector;
static bool eq(vector<int> a, vector<int> b) {
    std::sort(a.begin(), a.end());
    std::sort(b.begin(), b.end());
    return a == b;
}
//__USER__
int main() {
    { vector<int> a{1,2,2,1}, b{2,2}; if (!eq(intersect(a,b), {2,2})) { std::puts("case1"); return 1; } }
    { vector<int> a{4,9,5}, b{9,4,9,8,4}; if (!eq(intersect(a,b), {4,9})) { std::puts("case2"); return 1; } }
    { vector<int> a{1,1}, b{2,2}; if (!eq(intersect(a,b), {})) { std::puts("case3"); return 1; } }
    { vector<int> a{1,2,2,1}, b{2}; if (!eq(intersect(a,b), {2})) { std::puts("case4"); return 1; } }
    { vector<int> a{3,1,2}, b{1,1,2,3,3}; if (!eq(intersect(a,b), {1,2,3})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because the result respects multiplicity, model each array as a multiset. Count values from `nums1` in a hash map, then walk `nums2`: emit a value only while its remaining count is positive, decrementing on each emission. The decrement caps the number of copies at `min(count in nums1, count in nums2)`, exactly the multiset intersection. O(n + m) time, O(min(n,m)) space.
