## challenge: Single Element in a Sorted Array
tags: binary-search, array
track: faang
difficulty: medium

You are given a sorted array where every element appears exactly twice, except for one element that appears only once. Return the single element. You must run in O(log n) time and O(1) space.

Constraints: `1 <= nums.length <= 10^5`, `nums.length` is odd, `0 <= nums[i] <= 10^5`, `nums` is sorted ascending.

Example: `nums = [1,1,2,3,3,4,4,8,8]` → `2`. Example: `nums = [3,3,7,7,10,11,11]` → `10`. Example: `nums = [1]` → `1`.

hint: Before the single element, each pair starts at an even index; after it, that pairing is shifted by one.
hint: Snap `mid` down to an even index. If `nums[mid] == nums[mid + 1]`, the singleton is to the right of this intact pair; otherwise it is at `mid` or to its left.
hint: Because the length is odd, the window always narrows to exactly the single element — no separate final check is needed.

```cpp
// starter
#include <vector>
int singleNonDuplicate(std::vector<int>& nums);
```

```cpp
int singleNonDuplicate(std::vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid % 2 == 1) --mid;               // align to the even index of a pair
        if (nums[mid] == nums[mid + 1]) lo = mid + 2;
        else hi = mid;
    }
    return nums[lo];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,1,2,3,3,4,4,8,8}; if (singleNonDuplicate(n) != 2)  { std::puts("case1"); return 1; } }
    { vector<int> n{3,3,7,7,10,11,11};  if (singleNonDuplicate(n) != 10) { std::puts("case2"); return 1; } }
    { vector<int> n{1};                 if (singleNonDuplicate(n) != 1)  { std::puts("case3"); return 1; } }
    { vector<int> n{1,1,2};             if (singleNonDuplicate(n) != 2)  { std::puts("case4"); return 1; } }
    { vector<int> n{0,0,1,1,2};         if (singleNonDuplicate(n) != 2)  { std::puts("case5"); return 1; } }
    { vector<int> n{2,3,3,4,4};         if (singleNonDuplicate(n) != 2)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Exploit index parity. To the left of the singleton, each matching pair occupies indices `(even, odd)`; to the right of it, the pattern flips. Force `mid` to the even index of its pair: if it matches its right neighbor, the intact pair means the anomaly lies further right, so jump `lo` past both; otherwise the anomaly is at or before `mid`. The odd length guarantees convergence onto the unique element. O(log n) time, O(1) space.
