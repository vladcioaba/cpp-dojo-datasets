## challenge: Permutation Sequence
tags: backtracking, math, recursion
track: faang
difficulty: hard

The set `[1, 2, ..., n]` has `n!` distinct permutations. If all of them are listed and labelled in strictly increasing (lexicographic) order, return the `k`-th permutation sequence (1-indexed) as a string.

Constraints: `1 <= n <= 9`, `1 <= k <= n!`.

Example: `n = 3, k = 3` -> `"213"` (the order is `123, 132, 213, 231, 312, 321`). Example: `n = 4, k = 9` -> `"2314"`. Example: `n = 1, k = 1` -> `"1"`.

hint: Fixing the first digit determines a contiguous block of `(n-1)!` permutations, so `k` reveals directly which digit must lead.
hint: Work in the factorial number system: repeatedly divide the zero-based index by `(n-1)!`, `(n-2)!`, ... to select each next unused digit.
hint: Remove every chosen digit from the pool so later divisions index into the digits that remain.

```cpp
// starter
#include <string>
std::string getPermutation(int n, int k);
```

```cpp
std::string getPermutation(int n, int k) {
    std::vector<int> fact(n + 1, 1);
    for (int i = 1; i <= n; ++i) fact[i] = fact[i - 1] * i;
    std::vector<int> nums;
    for (int i = 1; i <= n; ++i) nums.push_back(i);
    --k;
    std::string res;
    for (int i = n; i >= 1; --i) {
        int idx = k / fact[i - 1];
        k %= fact[i - 1];
        res.push_back(char('0' + nums[idx]));
        nums.erase(nums.begin() + idx);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <algorithm>
using std::string;
//__USER__
int main() {
    for (int n = 1; n <= 7; ++n) {
        string s;
        for (int i = 1; i <= n; ++i) s.push_back(char('0' + i));
        int f = 1;
        for (int i = 1; i <= n; ++i) f *= i;
        for (int k = 1; k <= f; ++k) {
            if (getPermutation(n, k) != s) {
                std::printf("fail n=%d k=%d\n", n, k); return 1;
            }
            std::next_permutation(s.begin(), s.end());
        }
    }
    if (getPermutation(9, 1) != "123456789")      { std::puts("big1"); return 1; }
    if (getPermutation(9, 362880) != "987654321") { std::puts("big2"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Instead of generating permutations one by one, decode `k` (made zero-based) directly. The leading digit is index `k / (n-1)!` into the sorted remaining digits; subtract that block off with a modulo and repeat for `(n-2)!`, and so on. Removing each chosen digit keeps the pool sorted. This runs in O(n^2) time (the erase dominates) with no search at all; the harness cross-checks every permutation for `n <= 7` against `std::next_permutation`.
