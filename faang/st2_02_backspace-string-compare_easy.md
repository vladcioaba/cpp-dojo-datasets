## challenge: Backspace String Compare
tags: stack, string, simulation
track: faang
difficulty: easy

Given two strings `s` and `t`, return `true` if they are equal after being typed into empty text editors, where `#` means a backspace character. A backspace on an already-empty text does nothing.

Constraints: `1 <= s.length, t.length <= 200`; `s` and `t` contain only lowercase English letters and `#`.

Example: `s = "ab#c", t = "ad#c"` → `true` (both become `"ac"`). Example: `s = "a#c", t = "b"` → `false` (`"c"` vs `"b"`).

hint: Simulate the typing: a normal character appends, and `#` deletes the most recent character.
hint: A stack (or a mutable string used as one) reproduces the editor exactly — push letters, pop on `#`.
hint: Build the final text for each string independently, then compare the two results.

```cpp
// starter
#include <string>
bool backspaceCompare(std::string s, std::string t);
```

```cpp
bool backspaceCompare(std::string s, std::string t) {
    auto build = [](const std::string& x) {
        std::string st;
        for (char c : x) {
            if (c == '#') { if (!st.empty()) st.pop_back(); }
            else st.push_back(c);
        }
        return st;
    };
    return build(s) == build(t);
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (!backspaceCompare("ab#c", "ad#c")) { std::puts("case1"); return 1; }
    if (!backspaceCompare("ab##", "c#d#")) { std::puts("case2"); return 1; }
    if ( backspaceCompare("a#c", "b"))     { std::puts("case3"); return 1; }
    if (!backspaceCompare("a##c", "#a#c")) { std::puts("case4"); return 1; }
    if ( backspaceCompare("bxj##tw", "bxj###tw")) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Replay each string into a stack: push ordinary characters and, on `#`, pop the top if one exists (a backspace on empty text is a no-op). The final stack contents are exactly the text that would appear in the editor. Building both and comparing costs O(s + t) time and O(s + t) space. A more advanced O(1)-space variant walks both strings from the right, skipping characters that later backspaces delete.
