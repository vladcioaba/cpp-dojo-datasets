## challenge: Take K of Each Character From Left and Right
tags: string, hash-table, sliding-window
track: faang
difficulty: hard

You are given a string `s` consisting only of the characters `'a'`, `'b'`, and `'c'`, and a non-negative integer `k`. In each move you may take one character from either the leftmost or the rightmost end of `s` (removing it). Return the minimum number of moves needed so that you have taken at least `k` copies of each of the three characters, or `-1` if it is impossible.

Constraints: `1 <= s.length <= 10^5`, `s` consists of `'a'`, `'b'`, and `'c'`, `1 <= k <= s.length`.

Example: `s = "aabaaaacaabc", k = 2` → `8` (take the last three characters `b`, `c`, plus enough from the front to collect two of each; the untouched middle block has length `4`). Example: `s = "a", k = 1` → `-1` (there are no `b` or `c` at all).

hint: The characters you take always form a prefix and a suffix, leaving a single contiguous block untouched in the middle.

hint: Minimizing the number taken is the same as maximizing the length of that untouched middle block.

hint: The middle block is valid when, for every character, `count outside the block >= k` — equivalently the block may contain at most `total[c] - k` copies of character `c`. Find the longest such window; the answer is `n - windowLength`.

```cpp
// starter
#include <string>
int takeCharacters(std::string s, int k);
```

```cpp
int takeCharacters(std::string s, int k) {
    int total[3] = {0, 0, 0};
    for (char c : s) ++total[c - 'a'];
    if (total[0] < k || total[1] < k || total[2] < k) return -1;
    int allow[3] = {total[0] - k, total[1] - k, total[2] - k};
    int cnt[3] = {0, 0, 0};
    int left = 0, best = 0, n = s.size();
    for (int right = 0; right < n; ++right) {
        int c = s[right] - 'a';
        ++cnt[c];
        while (cnt[c] > allow[c]) {
            --cnt[s[left] - 'a'];
            ++left;
        }
        best = std::max(best, right - left + 1);
    }
    return n - best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <algorithm>
using std::string;
//__USER__
int main() {
    if (takeCharacters("aabaaaacaabc",2)!=8) { std::puts("case1"); return 1; }
    if (takeCharacters("a",1)!=-1) { std::puts("case2"); return 1; }
    if (takeCharacters("abc",1)!=3) { std::puts("case3"); return 1; }
    if (takeCharacters("aabbcc",1)!=4) { std::puts("case4"); return 1; }
    if (takeCharacters("cccccbbbaaa",1)!=5) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Since every move peels a character off one end, the taken characters are exactly a prefix plus a suffix, and what remains is one contiguous middle block. Taking fewer characters means keeping a longer block, so we maximize the middle window subject to leaving at least `k` of each character outside it — that is, the window may hold at most `total[c] - k` copies of each character `c`. If any character occurs fewer than `k` times, the goal is impossible and the answer is `-1`. Otherwise slide a window that respects all three caps, tracking its maximum length `best`; the minimum moves is `n - best`. Each character is visited by both pointers at most once, so this is O(n) time and O(1) space.
