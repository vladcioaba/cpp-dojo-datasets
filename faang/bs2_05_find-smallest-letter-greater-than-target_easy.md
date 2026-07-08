## challenge: Find Smallest Letter Greater Than Target
tags: binary-search, array
track: faang
difficulty: easy

Given a sorted array of lowercase `letters` and a character `target`, return the smallest letter in the array that is **strictly greater** than `target`. The letters wrap around: if no letter is greater than `target`, return `letters[0]`.

Constraints: `2 <= letters.length <= 10^4`, `letters[i]` is a lowercase English letter, `letters` is sorted in non-decreasing order, `letters` may contain duplicates.

Example: `letters = ['c','f','j'], target = 'a'` → `'c'`. Example: `target = 'c'` → `'f'`. Example: `target = 'j'` → `'c'` (wraps around).

hint: You want the first letter that is `> target`; the predicate "`letters[i] > target`" is false, false, ..., then true.
hint: Binary search for the leftmost index whose letter exceeds `target`, treating `letters[i] <= target` as "go right".
hint: If the search runs off the end, the wrap rule means the answer is `letters[0]` — take the index modulo the length.

```cpp
// starter
#include <vector>
char nextGreatestLetter(std::vector<char>& letters, char target);
```

```cpp
char nextGreatestLetter(std::vector<char>& letters, char target) {
    int lo = 0, hi = (int)letters.size();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (letters[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return letters[lo % letters.size()];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<char> l{'c','f','j'}; if (nextGreatestLetter(l, 'a') != 'c') { std::puts("case1"); return 1; } }
    { vector<char> l{'c','f','j'}; if (nextGreatestLetter(l, 'c') != 'f') { std::puts("case2"); return 1; } }
    { vector<char> l{'c','f','j'}; if (nextGreatestLetter(l, 'd') != 'f') { std::puts("case3"); return 1; } }
    { vector<char> l{'c','f','j'}; if (nextGreatestLetter(l, 'j') != 'c') { std::puts("case4"); return 1; } }
    { vector<char> l{'c','f','j'}; if (nextGreatestLetter(l, 'g') != 'j') { std::puts("case5"); return 1; } }
    { vector<char> l{'x','x','y','y'}; if (nextGreatestLetter(l, 'z') != 'x') { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is an upper-bound search: find the leftmost position whose letter is strictly greater than `target`. Push `lo` past every letter that is `<= target`; the loop settles on the first larger letter, or on `letters.size()` when none exists. Taking the index modulo the length applies the wrap-around rule cleanly, returning `letters[0]`. O(log n) time, O(1) space.
