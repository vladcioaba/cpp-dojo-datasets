## challenge: Fruit Into Baskets
tags: array, hash-table, sliding-window
track: faang
difficulty: medium

You are given an integer array `fruits` where `fruits[i]` is the type of fruit the `i`-th tree produces. You have two baskets, each of which can hold only a single type of fruit (in unlimited quantity). Starting from any tree you pick exactly one fruit from every tree moving right, and stop as soon as you reach a tree whose fruit fits in neither basket. Return the maximum number of fruits you can pick — equivalently, the length of the longest contiguous subarray containing at most two distinct values.

Constraints: `1 <= fruits.length <= 10^5`, `0 <= fruits[i] < fruits.length`.

Example: `fruits = [1,2,1]` → `3`. Example: `fruits = [0,1,2,2]` → `3` (`[1,2,2]`). Example: `fruits = [1,2,3,2,2]` → `4` (`[2,3,2,2]`).

hint: Strip away the story: you want the longest window with no more than two distinct fruit types.
hint: Keep a count per fruit type in the current window; the window is valid while it holds at most two distinct keys.
hint: When a third type appears, shrink from the left, decrementing counts and erasing a type when its count hits zero, until only two types remain.

```cpp
// starter
#include <vector>
int totalFruit(std::vector<int>& fruits);
```

```cpp
int totalFruit(std::vector<int>& fruits) {
    std::unordered_map<int,int> cnt;
    int left = 0, best = 0;
    for (int right = 0; right < (int)fruits.size(); ++right) {
        ++cnt[fruits[right]];
        while ((int)cnt.size() > 2) {
            int f = fruits[left++];
            if (--cnt[f] == 0) cnt.erase(f);
        }
        best = std::max(best, right - left + 1);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> f{1,2,1}; if (totalFruit(f)!=3) { std::puts("case1"); return 1; } }
    { vector<int> f{0,1,2,2}; if (totalFruit(f)!=3) { std::puts("case2"); return 1; } }
    { vector<int> f{1,2,3,2,2}; if (totalFruit(f)!=4) { std::puts("case3"); return 1; } }
    { vector<int> f{3,3,3,1,2,1,1,2,3,3,4}; if (totalFruit(f)!=5) { std::puts("case4"); return 1; } }
    { vector<int> f{5}; if (totalFruit(f)!=1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is the classic "longest subarray with at most two distinct elements". Slide a window while maintaining a hash map of value → count within it. Extend the right edge and, whenever the map holds more than two keys, advance the left edge, decrementing counts and erasing any key that reaches zero. The largest window width seen is the answer. Every index is added and removed once, so O(n) time and O(1) extra space (the map holds at most three keys at any instant).
