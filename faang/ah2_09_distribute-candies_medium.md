## challenge: Distribute Candies
tags: array, hash-table, greedy, arrays-hashing
track: faang
difficulty: medium

Alice has `n` candies where `candyType[i]` is the type of the `i`-th candy. The doctor advises Alice to eat only `n / 2` of the candies (`n` is even). Return the maximum number of different types of candies she can eat if she eats exactly `n / 2` candies.

Constraints: `n == candyType.length`, `2 <= n <= 10^4`, `n` is even, `-10^5 <= candyType[i] <= 10^5`.

Example: `candyType = [1,1,2,2,3,3]` → `3` (she can eat one of each of the 3 types). Example: `candyType = [1,1,2,3]` → `2` (she eats 2 candies; at best 2 distinct types). Example: `candyType = [6,6,6,6]` → `1`.

hint: She can eat at most `n / 2` candies, so the answer can never exceed `n / 2`.
hint: The number of distinct types is also an upper bound, since eating a type more than once wastes an opportunity for variety.
hint: The answer is the smaller of the distinct-type count and `n / 2`; a hash set gives the distinct count.

```cpp
// starter
#include <vector>
int distributeCandies(std::vector<int>& candyType);
```

```cpp
int distributeCandies(std::vector<int>& candyType) {
    std::unordered_set<int> kinds(candyType.begin(), candyType.end());
    return (int)std::min(kinds.size(), candyType.size() / 2);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_set>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> c{1,1,2,2,3,3}; if (distributeCandies(c) != 3) { std::puts("case1"); return 1; } }
    { vector<int> c{1,1,2,3}; if (distributeCandies(c) != 2) { std::puts("case2"); return 1; } }
    { vector<int> c{6,6,6,6}; if (distributeCandies(c) != 1) { std::puts("case3"); return 1; } }
    { vector<int> c{1,2,3,4,5,6}; if (distributeCandies(c) != 3) { std::puts("case4"); return 1; } }
    { vector<int> c{-1,-2,-3,-4}; if (distributeCandies(c) != 2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Two ceilings bound the answer. Alice may eat only `n/2` candies, and she can experience at most as many distinct types as actually exist. To maximize variety she greedily eats a new type whenever possible, so the achievable maximum is exactly `min(distinct types, n/2)`. A hash set counts distinct types in O(n) time and O(n) space.
