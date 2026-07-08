## challenge: Palindrome Partitioning
tags: backtracking, string, dynamic-programming
track: faang
difficulty: medium

Given a string `s`, partition `s` such that every substring of the partition is a palindrome. Return all possible palindrome partitionings of `s`. Each partitioning is the ordered list of substrings whose concatenation is `s`. Return the partitionings in any order.

Constraints: `1 <= s.length <= 16`, `s` consists of lowercase English letters.

Example: `s = "aab"` -> `[["a","a","b"],["aa","b"]]`. Example: `s = "a"` -> `[["a"]]`. Example: `s = "aba"` -> `[["a","b","a"],["aba"]]`.

hint: At each step you choose the next chunk of the string — it must be a palindrome — and then recurse on the rest.
hint: Backtrack over the cut position: from index `start`, try every `end >= start` where `s[start..end]` is a palindrome, take that prefix, and recurse from `end + 1`.
hint: When `start` reaches the end of the string, the current list of chunks is one complete valid partition — record a copy. Checking a palindrome is a simple two-pointer scan.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::vector<std::string>> partition(std::string s);
```

```cpp
std::vector<std::vector<std::string>> partition(std::string s) {
    std::vector<std::vector<std::string>> res;
    std::vector<std::string> cur;
    int n = (int)s.size();
    auto isPal = [&](int l, int r) {
        while (l < r) { if (s[l] != s[r]) return false; ++l; --r; }
        return true;
    };
    std::function<void(int)> dfs = [&](int start) {
        if (start == n) { res.push_back(cur); return; }
        for (int end = start; end < n; ++end) {
            if (isPal(start, end)) {
                cur.push_back(s.substr(start, end - start + 1));
                dfs(end + 1);
                cur.pop_back();
            }
        }
    };
    dfs(0);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <functional>
#include <algorithm>
using std::vector;
using std::string;
static vector<vector<string>> canonOuter(vector<vector<string>> g) {
    std::sort(g.begin(), g.end()); // inner lists are ordered (must concatenate to s); sort only outer
    return g;
}
//__USER__
int main() {
    {
        vector<vector<string>> want = {{"a","a","b"},{"aa","b"}};
        if (canonOuter(partition("aab")) != canonOuter(want)) { std::puts("case1"); return 1; }
    }
    {
        vector<vector<string>> want = {{"a"}};
        if (canonOuter(partition("a")) != canonOuter(want)) { std::puts("case2"); return 1; }
    }
    {
        vector<vector<string>> want = {{"a","b","a"},{"aba"}};
        if (canonOuter(partition("aba")) != canonOuter(want)) { std::puts("case3"); return 1; }
    }
    {
        // "aaa" -> 4 partitions, and every chunk must be a palindrome equal to a run of 'a'
        auto got = partition("aaa");
        vector<vector<string>> want = {{"a","a","a"},{"a","aa"},{"aa","a"},{"aaa"}};
        if (canonOuter(got) != canonOuter(want)) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Backtracking chooses the next palindromic prefix at each cut point and recurses on the remaining suffix, undoing the choice on the way back. A two-pointer check confirms a chunk is a palindrome in linear time. Every path that consumes the whole string yields a valid partition. In the worst case (a string of identical characters) there are `2^(n-1)` partitions, so time is exponential, O(n * 2^n), with O(n) recursion depth.
