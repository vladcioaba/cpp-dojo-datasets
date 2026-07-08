## challenge: Sort Array By Parity II
tags: two-pointers, array, sorting
track: faang
difficulty: easy

Given an array `nums` of non-negative integers, half of which are even and half odd, rearrange it so that whenever `i` is even `nums[i]` is even, and whenever `i` is odd `nums[i]` is odd. Return any array that satisfies this condition.

Constraints: `2 <= nums.length <= 2*10^4`, `nums.length` is even, half the elements are even and half are odd, `0 <= nums[i] <= 1000`.

Example: `nums = [4,2,5,7]` → `[4,5,2,7]` (even values at even indices, odd values at odd indices). Example: `nums = [2,3]` → `[2,3]`.

hint: You need two independent write positions: one walking the even indices `0,2,4,...` and one walking the odd indices `1,3,5,...`.
hint: A single pass over the input can dispatch each value to the correct stream based on its parity.
hint: Send every even value to the next even slot and every odd value to the next odd slot; the counts are guaranteed to match up.

```cpp
// starter
#include <vector>
std::vector<int> sortArrayByParityII(std::vector<int>& nums);
```

```cpp
std::vector<int> sortArrayByParityII(std::vector<int>& nums) {
    int n = (int)nums.size();
    std::vector<int> res(n);
    int i = 0, j = 1;
    for (int x : nums) {
        if (x % 2 == 0) { res[i] = x; i += 2; }
        else { res[j] = x; j += 2; }
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
    { vector<int> n{4,2,5,7}; if (sortArrayByParityII(n) != vector<int>{4,5,2,7}) { std::puts("case1"); return 1; } }
    { vector<int> n{2,3};     if (sortArrayByParityII(n) != vector<int>{2,3}) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2};     if (sortArrayByParityII(n) != vector<int>{2,1}) { std::puts("case3"); return 1; } }
    { vector<int> n{3,1,4,2}; if (sortArrayByParityII(n) != vector<int>{4,3,2,1}) { std::puts("case4"); return 1; } }
    { vector<int> n{0,0,0,1,1,1}; if (sortArrayByParityII(n) != vector<int>{0,1,0,1,0,1}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Keep two write cursors, `i` over even indices and `j` over odd indices, both stepping by two. Sweep the input once and route each value by its own parity: evens land at `res[i]` and advance `i`, odds land at `res[j]` and advance `j`. Since the array is guaranteed to hold equal numbers of even and odd values, both cursors fill their halves exactly. O(n) time and O(n) space for the output.
