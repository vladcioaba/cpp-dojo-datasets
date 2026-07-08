## challenge: Partition Labels
tags: greedy, hash-table, two-pointers, string
track: faang
difficulty: medium

Given a string `s`, partition it into as many parts as possible so that each letter appears in at most one part. Return a list of the part sizes, in order.

Constraints: `1 <= s.length <= 500`, `s` consists of lowercase English letters.

Example: `s = "ababcbacadefegdehijhklij"` → `[9,7,8]`. Example: `s = "eccbbbbdec"` → `[10]`.

hint: A part can only close once you have passed the last occurrence of every letter it contains.
hint: Precompute the final index of each letter in a single scan.
hint: Sweep left to right extending the current part's end to the farthest last-occurrence seen; cut when the cursor reaches that end.

```cpp
// starter
#include <vector>
#include <string>
std::vector<int> partitionLabels(std::string s);
```

```cpp
std::vector<int> partitionLabels(std::string s) {
    int last[26];
    for (int i = 0; i < (int)s.size(); ++i) last[s[i] - 'a'] = i;
    std::vector<int> out;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.size(); ++i) {
        end = std::max(end, last[s[i] - 'a']);
        if (i == end) { out.push_back(end - start + 1); start = i + 1; }
    }
    return out;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <algorithm>
using std::vector;
using std::string;
//__USER__
int main() {
    { string s = "ababcbacadefegdehijhklij"; auto r = partitionLabels(s);
      if (r != vector<int>({9,7,8})) { std::puts("case1"); return 1; } }
    { string s = "eccbbbbdec"; auto r = partitionLabels(s);
      if (r != vector<int>({10})) { std::puts("case2"); return 1; } }
    { string s = "a"; auto r = partitionLabels(s);
      if (r != vector<int>({1})) { std::puts("case3"); return 1; } }
    { string s = "abcabc"; auto r = partitionLabels(s);
      if (r != vector<int>({6})) { std::puts("case4"); return 1; } }
    { string s = "abcdef"; auto r = partitionLabels(s);
      if (r != vector<int>({1,1,1,1,1,1})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Record each letter's last index. Sweeping the string, keep the current part's end at the maximum last-occurrence among letters seen so far; the moment the cursor equals that end, every letter inside has been fully consumed and the part can close greedily. This yields the maximal number of parts. O(n) time, O(1) extra space.
