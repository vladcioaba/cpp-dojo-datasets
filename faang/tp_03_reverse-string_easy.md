## challenge: Reverse String
tags: two-pointers, string
track: faang
difficulty: easy

Write a function that reverses a string. The input is given as an array of characters `s`, and you must do it in place with O(1) extra memory (no allocating a second array).

Constraints: `1 <= s.length <= 10^5`, `s[i]` is a printable ASCII character.

Example: `s = ['h','e','l','l','o']` → `['o','l','l','e','h']`. Example: `s = ['H','a','n','n','a','h']` → `['h','a','n','n','a','H']`.

hint: Reversing means the first character trades places with the last, the second with the second-to-last, and so on.
hint: Two pointers, one at each end, swapping and stepping toward the center, touch every pair exactly once.
hint: While the left index is below the right index, swap `s[left]` and `s[right]`, then move left up and right down.

```cpp
// starter
#include <vector>
void reverseString(std::vector<char>& s);
```

```cpp
void reverseString(std::vector<char>& s) {
    int lo = 0, hi = (int)s.size() - 1;
    while (lo < hi) {
        std::swap(s[lo], s[hi]);
        ++lo;
        --hi;
    }
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <utility>
using std::vector;
//__USER__
int main() {
    { vector<char> s{'h','e','l','l','o'}; reverseString(s); if (s != vector<char>{'o','l','l','e','h'}) { std::puts("case1"); return 1; } }
    { vector<char> s{'H','a','n','n','a','h'}; reverseString(s); if (s != vector<char>{'h','a','n','n','a','H'}) { std::puts("case2"); return 1; } }
    { vector<char> s{'a'}; reverseString(s); if (s != vector<char>{'a'}) { std::puts("case3"); return 1; } }
    { vector<char> s{'a','b'}; reverseString(s); if (s != vector<char>{'b','a'}) { std::puts("case4"); return 1; } }
    { vector<char> s{'x','y','z','1','2'}; reverseString(s); if (s != vector<char>{'2','1','z','y','x'}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A reversal is a set of independent swaps between mirror-image positions. Put one pointer at the start and one at the end and swap the characters they reference, then walk the pointers toward each other. When they meet (or cross) the whole array has been mirrored. Exactly `n/2` swaps run in O(n) time using O(1) extra space.
