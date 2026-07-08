## challenge: Letter Case Permutation
tags: backtracking, string, bit-manipulation
track: faang
difficulty: easy

Given a string `s`, transform each letter individually to be lowercase or uppercase to create a new string. Digits (and any non-letter characters) are left unchanged. Return a list of every string that can be created this way, in any order.

Constraints: `1 <= s.length <= 12`, `s` consists of lowercase and uppercase English letters and digits.

Example: `s = "a1b2"` -> `["a1b2","a1B2","A1b2","A1B2"]`. Example: `s = "3z4"` -> `["3z4","3Z4"]`. Example: `s = "12345"` -> `["12345"]`.

hint: Only letters branch; each letter offers two choices (lower / upper) while digits are fixed.
hint: Backtrack character by character — append the fixed character for digits and try both cases for letters.
hint: The number of results is `2^(number of letters)`.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> letterCasePermutation(std::string s);
```

```cpp
std::vector<std::string> letterCasePermutation(std::string s) {
    std::vector<std::string> res;
    std::string cur;
    std::function<void(int)> dfs = [&](int i) {
        if (i == (int)s.size()) { res.push_back(cur); return; }
        char c = s[i];
        if (std::isalpha((unsigned char)c)) {
            cur.push_back((char)std::tolower((unsigned char)c));
            dfs(i + 1);
            cur.pop_back();
            cur.push_back((char)std::toupper((unsigned char)c));
            dfs(i + 1);
            cur.pop_back();
        } else {
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
#include <cctype>
#include <algorithm>
using std::vector;
using std::string;
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
static vector<string> reference(const string& s) {
    vector<int> pos;
    for (int i = 0; i < (int)s.size(); ++i)
        if (std::isalpha((unsigned char)s[i])) pos.push_back(i);
    int k = (int)pos.size();
    vector<string> out;
    for (int mask = 0; mask < (1 << k); ++mask) {
        string t = s;
        for (int j = 0; j < k; ++j)
            t[pos[j]] = (mask & (1 << j)) ? (char)std::toupper((unsigned char)t[pos[j]])
                                          : (char)std::tolower((unsigned char)t[pos[j]]);
        out.push_back(t);
    }
    return out;
}
//__USER__
int main() {
    const char* tests[] = {"a1b2", "3z4", "12345", "C", "aB", "0"};
    for (auto* p : tests) {
        string s = p;
        if (canon(letterCasePermutation(s)) != canon(reference(s))) {
            std::printf("fail:%s\n", p); return 1;
        }
    }
    { string s = "a1b2";
      if (letterCasePermutation(s).size() != 4) { std::puts("size"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Walk the string once; at a letter, branch into the lowercase and uppercase variants, and at a non-letter simply carry it through. Each leaf of the decision tree is one full string. With `L` letters there are `2^L` results, each of length `n`, giving O(n * 2^L) time and O(n) recursion depth. The harness cross-checks against a bitmask enumeration over the letter positions.
