## challenge: Number of Good Pairs
tags: array, hash-table, counting, arrays-hashing
track: faang
difficulty: easy

Given an array of integers `nums`, return the number of good pairs. A pair `(i, j)` is called good if `nums[i] == nums[j]` and `i < j`.

Constraints: `1 <= nums.length <= 100`, `1 <= nums[i] <= 100`.

Example: `nums = [1,2,3,1,1,3]` → `4` (the good pairs are (0,3), (0,4), (3,4), (2,5)). Example: `nums = [1,1,1,1]` → `6` (every pair is good). Example: `nums = [1,2,3]` → `0`.

hint: A value that appears `c` times contributes `c*(c-1)/2` good pairs; you could count occurrences then sum that formula.
hint: Alternatively, count in a single pass: before recording a new occurrence of a value, every earlier occurrence forms a good pair with it.
hint: Keep a running count per value in a hash map, add that count to the answer, then increment it.

```cpp
// starter
#include <vector>
int numIdenticalPairs(std::vector<int>& nums);
```

```cpp
int numIdenticalPairs(std::vector<int>& nums) {
    std::unordered_map<int, int> cnt;
    int pairs = 0;
    for (int x : nums) {
        pairs += cnt[x];
        cnt[x]++;
    }
    return pairs;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,3,1,1,3}; if (numIdenticalPairs(n) != 4) { std::puts("case1"); return 1; } }
    { vector<int> n{1,1,1,1}; if (numIdenticalPairs(n) != 6) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3}; if (numIdenticalPairs(n) != 0) { std::puts("case3"); return 1; } }
    { vector<int> n{7}; if (numIdenticalPairs(n) != 0) { std::puts("case4"); return 1; } }
    { vector<int> n{5,5,5}; if (numIdenticalPairs(n) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Every good pair joins a new occurrence of a value with each earlier occurrence of the same value. So as you scan, the number of times you have already seen the current value is exactly the number of fresh good pairs it creates. Add that running count to the answer before incrementing it. This one-pass counting is O(n) time and O(u) space for `u` distinct values, avoiding the O(n^2) pairwise scan.
