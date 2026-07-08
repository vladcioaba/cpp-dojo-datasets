## challenge: Partition Labels
tags: two-pointers, greedy, string, hash-table
track: faang
difficulty: hard

You are given a string `s`. Partition it into as many parts as possible so that each letter appears in at most one part, and return a list of the sizes of these parts (in order). The concatenation of the parts, in order, must equal `s`.

Constraints: `1 <= s.length <= 500`, `s` consists of lowercase English letters.

Example: `s = "ababcbacadefegdehijhklij"` → `[9,7,8]`. The partition is `"ababcbaca"`, `"defegde"`, `"hijhklij"`. Example: `s = "eccbbbbdec"` → `[10]`.

hint: A part cannot end before the last occurrence of any letter it already contains, so the last position of each character is the key fact.
hint: Precompute, for every letter, the index of its final appearance in the string.
hint: Sweep with two pointers, extending the current part's end to the furthest last-occurrence seen so far; when the scan index reaches that end, the part closes and its length is recorded.

```cpp
// starter
#include <vector>
#include <string>
std::vector<int> partitionLabels(std::string s);
```

```cpp
std::vector<int> partitionLabels(std::string s) {
    int last[26] = {0};
    for (int i = 0; i < (int)s.size(); ++i)
        last[s[i] - 'a'] = i;
    std::vector<int> res;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.size(); ++i) {
        if (last[s[i] - 'a'] > end) end = last[s[i] - 'a'];
        if (i == end) {
            res.push_back(end - start + 1);
            start = i + 1;
        }
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
using std::vector;
using std::string;
//__USER__
int main() {
    if (partitionLabels("ababcbacadefegdehijhklij") != vector<int>{9,7,8}) { std::puts("case1"); return 1; }
    if (partitionLabels("eccbbbbdec") != vector<int>{10}) { std::puts("case2"); return 1; }
    if (partitionLabels("a") != vector<int>{1}) { std::puts("case3"); return 1; }
    if (partitionLabels("abcdef") != vector<int>{1,1,1,1,1,1}) { std::puts("case4"); return 1; }
    if (partitionLabels("aabb") != vector<int>{2,2}) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** First record the last index at which each letter occurs. Then sweep the string with a growing window: `start` marks the current part's beginning, and `end` tracks the furthest last-occurrence among all letters seen since `start`. Whenever the scan index equals `end`, every letter inside the window is fully contained, so the part can be cut greedily and its length appended. Two linear passes (one to compute last positions, one to sweep) give O(n) time and O(1) extra space (a fixed 26-entry table).
