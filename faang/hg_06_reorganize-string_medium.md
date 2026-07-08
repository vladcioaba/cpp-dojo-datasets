## challenge: Reorganize String
tags: heap, greedy, string, hash-table
track: faang
difficulty: medium

Given a string `s`, rearrange its characters so that no two adjacent characters are the same, and return any such rearrangement. If it is impossible, return the empty string.

Constraints: `1 <= s.length <= 500`, `s` consists of lowercase English letters.

Example: `s = "aab"` → `"aba"` (any valid arrangement is accepted). Example: `s = "aaab"` → `""` (impossible).

hint: If any character appears more than `(n + 1) / 2` times it cannot be separated — the answer is impossible.
hint: Otherwise repeatedly place the two currently most frequent characters, which are never equal to each other.
hint: A max-heap keyed on remaining count lets you always pull the two most frequent letters; push each back with its count decremented while it is still positive.

```cpp
// starter
#include <string>
std::string reorganizeString(std::string s);
```

```cpp
std::string reorganizeString(std::string s) {
    int cnt[26] = {0};
    for (char c : s) ++cnt[c - 'a'];
    std::priority_queue<std::pair<int, char>> pq;
    for (int i = 0; i < 26; ++i) if (cnt[i]) pq.push({cnt[i], char('a' + i)});
    std::string res;
    while (pq.size() >= 2) {
        auto a = pq.top(); pq.pop();
        auto b = pq.top(); pq.pop();
        res += a.second;
        res += b.second;
        if (--a.first > 0) pq.push(a);
        if (--b.first > 0) pq.push(b);
    }
    if (!pq.empty()) {
        if (pq.top().first > 1) return "";
        res += pq.top().second;
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <queue>
#include <utility>
#include <algorithm>
using std::string;
static bool goodReorg(const string& r, const string& orig) {
    int c[26] = {0};
    for (char ch : orig) ++c[ch - 'a'];
    int fmax = 0;
    for (int i = 0; i < 26; ++i) fmax = std::max(fmax, c[i]);
    bool possible = fmax <= (int)(orig.size() + 1) / 2;
    if (!possible) return r.empty();
    if (r.size() != orig.size()) return false;
    int d[26] = {0};
    for (char ch : r) ++d[ch - 'a'];
    for (int i = 0; i < 26; ++i) if (c[i] != d[i]) return false;
    for (size_t i = 1; i < r.size(); ++i) if (r[i] == r[i - 1]) return false;
    return true;
}
//__USER__
int main() {
    { string s = "aab";    if (!goodReorg(reorganizeString(s), s)) { std::puts("case1"); return 1; } }
    { string s = "aaab";   if (!goodReorg(reorganizeString(s), s)) { std::puts("case2"); return 1; } }
    { string s = "a";      if (!goodReorg(reorganizeString(s), s)) { std::puts("case3"); return 1; } }
    { string s = "aaabbc"; if (!goodReorg(reorganizeString(s), s)) { std::puts("case4"); return 1; } }
    { string s = "vvvlo";  if (!goodReorg(reorganizeString(s), s)) { std::puts("case5"); return 1; } }
    { string s = "aaaabc"; if (!goodReorg(reorganizeString(s), s)) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A valid arrangement exists exactly when no letter's count exceeds `(n + 1) / 2`. Greedily emitting the two most frequent remaining letters each step guarantees they differ, so no adjacency is violated; a max-heap on counts supplies them in O(log 26) per pop. If a single letter is left with count greater than one, it is forced to touch itself and the task is impossible. O(n log 26) time, O(1) extra space.
