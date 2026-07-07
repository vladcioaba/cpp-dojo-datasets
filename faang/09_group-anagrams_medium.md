## challenge: Group Anagrams
tags: hash-table, string, sorting, arrays-hashing
track: faang
difficulty: medium

Given an array of strings `strs`, group the anagrams together. Two strings are anagrams if one is a rearrangement of the other. Return the groups in any order, and the strings within each group in any order.

Constraints: `1 <= strs.length <= 10^4`, `0 <= strs[i].length <= 100`, lowercase English letters.

Example: `["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"],["tan","nat"],["bat"]]`. Example: `[""]` → `[[""]]`.

hint: Anagrams share an invariant — a canonical form that is identical for every word in a group.
hint: Sorting a word's letters (or a 26-letter count signature) yields a key you can group on with a hash map.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs);
```

```cpp
std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
    std::unordered_map<std::string, std::vector<std::string>> groups;
    for (auto& s : strs) {
        std::string key = s;
        std::sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    std::vector<std::vector<std::string>> out;
    for (auto& [k, v] : groups) out.push_back(std::move(v));
    return out;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using std::vector;
using std::string;
static vector<vector<string>> canon(vector<vector<string>> g) {
    for (auto& row : g) std::sort(row.begin(), row.end());
    std::sort(g.begin(), g.end());
    return g;
}
//__USER__
int main() {
    {
        vector<string> in{"eat","tea","tan","ate","nat","bat"};
        auto got = canon(groupAnagrams(in));
        vector<vector<string>> want = canon({{"eat","tea","ate"},{"tan","nat"},{"bat"}});
        if (got != want) { std::puts("case1"); return 1; }
    }
    {
        vector<string> in{""};
        auto got = canon(groupAnagrams(in));
        vector<vector<string>> want = {{""}};
        if (got != want) { std::puts("case2"); return 1; }
    }
    {
        vector<string> in{"a"};
        auto got = canon(groupAnagrams(in));
        vector<vector<string>> want = {{"a"}};
        if (got != want) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Map each string to a canonical key — its letters sorted — and bucket the originals by that key in a hash map, so anagrams collide into the same group. For n strings of length k it is O(n*k log k) time (a counting-based key drops it to O(n*k)); O(n*k) space.
