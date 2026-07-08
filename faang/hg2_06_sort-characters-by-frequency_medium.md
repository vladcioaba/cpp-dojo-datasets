## challenge: Sort Characters By Frequency
tags: heap, priority-queue, hash-table, sorting

track: faang
difficulty: medium

Given a string `s`, rearrange its characters so that they appear in order of decreasing frequency and return the resulting string. All occurrences of a given character must stay together. If several arrangements are valid (for example when characters tie in frequency), any of them is accepted.

Constraints: `1 <= s.length <= 5*10^5`, `s` consists of upper/lower-case English letters and digits.

Example: `s = "tree"` → `"eert"` (`'e'` appears twice, `'r'` and `'t'` once each; `"eetr"` is also valid). Example: `s = "cccaaa"` → `"cccaaa"` (or `"aaaccc"`). Example: `s = "Aabb"` → `"bbAa"` (or `"bbaA"`).

hint: The only thing that matters is how often each character occurs, so start by counting.
hint: Emit characters from most frequent to least — a max-heap keyed on frequency does exactly this.
hint: When you pop a `(count, char)` pair, append that character `count` times to the output.

```cpp
// starter
#include <string>
std::string frequencySort(std::string s);
```

```cpp
std::string frequencySort(std::string s) {
    std::unordered_map<char, int> cnt;
    for (char c : s) cnt[c]++;
    std::priority_queue<std::pair<int, char>> pq;
    for (auto& [c, n] : cnt) pq.push({n, c});
    std::string res;
    res.reserve(s.size());
    while (!pq.empty()) {
        auto [n, c] = pq.top(); pq.pop();
        res.append(n, c);
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
#include <unordered_map>
using std::string;
// Accept any grouping where each character is contiguous and run lengths
// are non-increasing, and the multiset of characters is preserved.
static bool valid(const string& s, const string& r) {
    if (s.size() != r.size()) return false;
    int cs[256] = {0}, cr[256] = {0};
    for (unsigned char c : s) cs[c]++;
    for (unsigned char c : r) cr[c]++;
    for (int i = 0; i < 256; ++i) if (cs[i] != cr[i]) return false;
    int prev = 1 << 30;
    for (size_t i = 0; i < r.size(); ) {
        size_t j = i;
        while (j < r.size() && r[j] == r[i]) ++j;
        int run = (int)(j - i);
        if (run != cr[(unsigned char)r[i]]) return false;  // char must be fully contiguous
        if (run > prev) return false;                       // non-increasing frequency
        prev = run;
        i = j;
    }
    return true;
}
//__USER__
int main() {
    { string s = "tree";   if (!valid(s, frequencySort(s))) { std::puts("case1"); return 1; } }
    { string s = "cccaaa"; if (!valid(s, frequencySort(s))) { std::puts("case2"); return 1; } }
    { string s = "Aabb";   if (!valid(s, frequencySort(s))) { std::puts("case3"); return 1; } }
    { string s = "a";      if (!valid(s, frequencySort(s))) { std::puts("case4"); return 1; } }
    { string s = "2a554442f544asfasssffffasss"; if (!valid(s, frequencySort(s))) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Frequency is the only ordering criterion, so count each character, then drain a max-heap keyed on count, appending each character as many times as it occurs. The result groups identical characters together with run lengths in non-increasing order, which is exactly the required shape; ties may resolve either way, all equally valid. Counting is O(n); the heap holds at most one entry per distinct character, giving O(n + a log a) time for alphabet size `a` and O(n) space for the output.
