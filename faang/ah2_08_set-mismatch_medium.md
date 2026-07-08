## challenge: Set Mismatch
tags: array, hash-table, counting, arrays-hashing
track: faang
difficulty: medium

You have a set `s` that originally contained numbers from `1` to `n`. Due to an error, one number got duplicated to the value of another number in the set, leaving one number missing. Given the corrupted array `nums`, return an array `[duplicated, missing]`.

Constraints: `2 <= nums.length <= 10^4`, `1 <= nums[i] <= 10^4`. The array is a corrupted version of `1..n` where exactly one value appears twice and exactly one value is absent.

Example: `nums = [1,2,2,4]` → `[2,3]` (2 appears twice, 3 is missing). Example: `nums = [1,1]` → `[1,2]`. Example: `nums = [3,2,3,4,6,5]` → `[3,1]`.

hint: If you knew the count of each value in `1..n`, the duplicate has count 2 and the missing has count 0.
hint: Build a frequency array of size `n+1` in one pass.
hint: Then scan `1..n`: the value with count 2 is the duplicate and the value with count 0 is the missing one.

```cpp
// starter
#include <vector>
std::vector<int> findErrorNums(std::vector<int>& nums);
```

```cpp
std::vector<int> findErrorNums(std::vector<int>& nums) {
    int n = (int)nums.size();
    std::vector<int> cnt(n + 1, 0);
    for (int x : nums) cnt[x]++;
    int dup = -1, missing = -1;
    for (int i = 1; i <= n; ++i) {
        if (cnt[i] == 2) dup = i;
        else if (cnt[i] == 0) missing = i;
    }
    return {dup, missing};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,2,4}; auto r = findErrorNums(n); if (!(r.size()==2 && r[0]==2 && r[1]==3)) { std::puts("case1"); return 1; } }
    { vector<int> n{1,1}; auto r = findErrorNums(n); if (!(r.size()==2 && r[0]==1 && r[1]==2)) { std::puts("case2"); return 1; } }
    { vector<int> n{3,2,3,4,6,5}; auto r = findErrorNums(n); if (!(r.size()==2 && r[0]==3 && r[1]==1)) { std::puts("case3"); return 1; } }
    { vector<int> n{2,2}; auto r = findErrorNums(n); if (!(r.size()==2 && r[0]==2 && r[1]==1)) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** In a perfect `1..n` set every value has count exactly one. The corruption makes precisely one value appear twice and one value vanish. Tallying frequencies into a size-`n+1` array turns the problem into a scan: report the index with count two as the duplicate and the index with count zero as the missing number. O(n) time, O(n) space.
