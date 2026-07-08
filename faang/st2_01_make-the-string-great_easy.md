## challenge: Make The String Great
tags: stack, string
track: faang
difficulty: easy

Given a string `s` of lower- and upper-case English letters, a *good* string contains no two adjacent characters `s[i]` and `s[i+1]` that are the same letter in opposite cases (for example `s[i] = 'b'` and `s[i+1] = 'B'`, or vice versa). To make `s` good, repeatedly remove any such adjacent pair until none remain. Return the resulting good string; the answer is guaranteed to be unique.

Constraints: `1 <= s.length <= 100`; `s` consists only of lower- and upper-case English letters.

Example: `s = "leEeetcode"` → `"leetcode"` (remove the `eE` pair). Example: `s = "abBAcC"` → `""` (remove `bB`, then `aA`, then `cC`).

hint: Removing one pair can expose a new adjacent pair, so a left-to-right scan with a growing result works like undo.
hint: Keep the processed prefix on a stack; compare each new character against the top.
hint: Two letters are the same letter in opposite cases exactly when their ASCII codes differ by 32 — i.e. `(a ^ b) == 32`.

```cpp
// starter
#include <string>
std::string makeGood(std::string s);
```

```cpp
std::string makeGood(std::string s) {
    std::string st;
    for (char c : s) {
        if (!st.empty() && (st.back() ^ c) == 32) st.pop_back();
        else st.push_back(c);
    }
    return st;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (makeGood("leEeetcode") != "leetcode") { std::puts("case1"); return 1; }
    if (makeGood("abBAcC")     != "")         { std::puts("case2"); return 1; }
    if (makeGood("s")          != "s")        { std::puts("case3"); return 1; }
    if (makeGood("Pp")         != "")         { std::puts("case4"); return 1; }
    if (makeGood("mC")         != "mC")       { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Treat the processed prefix as a stack. For each incoming character, if it forms a bad pair with the current top — same letter, opposite case, detectable as `(top ^ c) == 32` — pop the top so both cancel; otherwise push it. Because cancelling a pair can bring two previously separated characters together, the stack naturally cascades those removals. Every character is pushed and popped at most once, giving O(n) time and O(n) space.
