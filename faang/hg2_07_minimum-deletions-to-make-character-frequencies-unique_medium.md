## challenge: Minimum Deletions to Make Character Frequencies Unique
tags: heap, priority-queue, greedy, hash-table

track: faang
difficulty: medium

A string is *good* if no two distinct characters occur the same number of times. Given a string `s`, return the minimum number of character deletions needed to make it good.

Constraints: `1 <= s.length <= 10^5`, `s` consists of lowercase English letters.

Example: `s = "aab"` → `0` (frequencies are `a:2, b:1`, already all distinct). Example: `s = "aaabbbcc"` → `2` (delete two to reach e.g. `a:3, b:2, c:1`). Example: `s = "abcabc"` → `3`.

hint: Only the multiset of frequencies matters; you want every frequency to be distinct with as few deletions as possible.
hint: Process frequencies largest-first and, whenever a value is already taken, keep decrementing it (counting a deletion each step) until it hits a free value or zero.
hint: A max-heap plus a set of already-used frequencies makes the "slide down to a free slot" step natural.

```cpp
// starter
#include <string>
int minDeletions(std::string s);
```

```cpp
int minDeletions(std::string s) {
    int freq[26] = {0};
    for (char c : s) freq[c - 'a']++;
    std::priority_queue<int> pq;
    for (int f : freq) if (f > 0) pq.push(f);
    std::unordered_set<int> used;
    int deletions = 0;
    while (!pq.empty()) {
        int cur = pq.top(); pq.pop();
        while (cur > 0 && used.count(cur)) { cur--; deletions++; }
        if (cur > 0) used.insert(cur);
    }
    return deletions;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <queue>
#include <unordered_set>
using std::string;
//__USER__
int main() {
    if (minDeletions("aab")      != 0) { std::puts("case1"); return 1; }
    if (minDeletions("aaabbbcc") != 2) { std::puts("case2"); return 1; }
    if (minDeletions("ceabaacb") != 2) { std::puts("case3"); return 1; }
    if (minDeletions("abcabc")   != 3) { std::puts("case4"); return 1; }
    if (minDeletions("a")        != 0) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Reduce the string to its 26 letter counts; the task becomes making those counts pairwise distinct with fewest decrements. Greedily handle the largest counts first: pop from a max-heap and, while the current value collides with one already claimed, decrement it (charging one deletion each time) until it reaches a free value or drops to zero. Tracking claimed values in a hash set makes each collision check O(1). The whole procedure is O(n) to count plus O(a log a) over the small alphabet, O(a) space; the greedy order guarantees the minimum total deletions.
