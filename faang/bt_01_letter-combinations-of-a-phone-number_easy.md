## challenge: Letter Combinations of a Phone Number
tags: backtracking, string, hash-table
track: faang
difficulty: easy

Given a string `digits` containing digits from `2-9`, return all the letter combinations that the digits could spell on a classic phone keypad. The mapping is the usual one: `2` -> `abc`, `3` -> `def`, `4` -> `ghi`, `5` -> `jkl`, `6` -> `mno`, `7` -> `pqrs`, `8` -> `tuv`, `9` -> `wxyz` (note `7` and `9` have four letters). Return the answer in any order.

Constraints: `0 <= digits.length <= 4`, each character of `digits` is in the range `2` to `9`. If `digits` is empty, return an empty list.

Example: `digits = "23"` -> `["ad","ae","af","bd","be","bf","cd","ce","cf"]`. Example: `digits = ""` -> `[]`. Example: `digits = "2"` -> `["a","b","c"]`.

hint: This is a Cartesian product — each digit contributes one letter drawn from its bucket, and you want every way to pick one letter per digit.
hint: Backtrack position by position: at digit `i`, loop over its letters, append one, recurse to digit `i+1`, then undo.
hint: The empty input is a special case — return `[]` rather than a list holding one empty string.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> letterCombinations(std::string digits);
```

```cpp
std::vector<std::string> letterCombinations(std::string digits) {
    if (digits.empty()) return {};
    static const std::vector<std::string> pad = {
        "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
    };
    std::vector<std::string> res;
    std::string cur;
    std::function<void(int)> dfs = [&](int i) {
        if (i == (int)digits.size()) { res.push_back(cur); return; }
        for (char c : pad[digits[i] - '0']) {
            cur.push_back(c);
            dfs(i + 1);
            cur.pop_back();
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
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    {
        auto got = letterCombinations("23");
        vector<string> want = {"ad","ae","af","bd","be","bf","cd","ce","cf"};
        if (canon(got) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        auto got = letterCombinations("");
        if (!got.empty()) { std::puts("case2"); return 1; }
    }
    {
        auto got = letterCombinations("2");
        vector<string> want = {"a","b","c"};
        if (canon(got) != canon(want)) { std::puts("case3"); return 1; }
    }
    {
        auto got = letterCombinations("79");
        if (got.size() != 16) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Each digit maps to a bucket of letters; the answer is every combination formed by choosing one letter per digit. Backtracking walks the digits left to right, appending a candidate letter and recursing, then undoing it. With `k` digits and at most four letters each, there are up to `4^k` combinations, giving O(4^k * k) time and O(k) recursion depth. Guard the empty string so it yields `[]`.
