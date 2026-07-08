## challenge: DI String Match
tags: two-pointers, string, greedy, math
track: faang
difficulty: easy

A permutation `perm` of the integers `0..n` can be encoded by a string `s` of length `n` where `s[i] == 'I'` means `perm[i] < perm[i+1]` and `s[i] == 'D'` means `perm[i] > perm[i+1]`. Given such a string `s`, reconstruct and return any permutation `perm` that matches it.

Constraints: `1 <= s.length <= 10^5`, `s[i]` is either `'I'` or `'D'`.

Example: `s = "IDID"` → `[0,4,1,3,2]`. Example: `s = "III"` → `[0,1,2,3]`. Example: `s = "DDI"` → `[3,2,0,1]`.

hint: You control the values, so keep the smallest and largest still-unused numbers ready at all times.
hint: An `'I'` demands the next value be bigger, which is safest satisfied by handing out the smallest remaining number; a `'D'` wants the largest.
hint: Track a low and a high bound over the unused range; consume from low on `'I'` and from high on `'D'`, then append whichever remains at the end.

```cpp
// starter
#include <string>
#include <vector>
std::vector<int> diStringMatch(std::string s);
```

```cpp
std::vector<int> diStringMatch(std::string s) {
    int lo = 0, hi = (int)s.size();
    std::vector<int> res;
    res.reserve(s.size() + 1);
    for (char c : s) {
        if (c == 'I') res.push_back(lo++);
        else res.push_back(hi--);
    }
    res.push_back(lo); // lo == hi now
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
using std::string;
using std::vector;
//__USER__
int main() {
    if (diStringMatch("IDID") != vector<int>{0,4,1,3,2}) { std::puts("case1"); return 1; }
    if (diStringMatch("III")  != vector<int>{0,1,2,3}) { std::puts("case2"); return 1; }
    if (diStringMatch("DDI")  != vector<int>{3,2,0,1}) { std::puts("case3"); return 1; }
    if (diStringMatch("D")    != vector<int>{1,0}) { std::puts("case4"); return 1; }
    if (diStringMatch("I")    != vector<int>{0,1}) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Maintain the unused range as `[lo, hi]`, starting at `[0, n]`. Because you get to choose the values, an `'I'` (needs a bigger successor) is always safe to satisfy with the current smallest, `lo`, and a `'D'` with the current largest, `hi`. Each choice shrinks the range by one and can never conflict with future comparisons. After processing all `n` characters, `lo == hi`, so append that final leftover value. O(n) time and O(n) space.
