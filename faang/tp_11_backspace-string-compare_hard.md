## challenge: Backspace String Compare
tags: two-pointers, string, stack
track: faang
difficulty: hard

Given two strings `s` and `t`, return `true` if they are equal after both are typed into empty text editors, where `'#'` represents a backspace character. A backspace on an already-empty text buffer does nothing. Solve it in O(n + m) time and O(1) extra space (no building the resulting strings).

Constraints: `1 <= s.length, t.length <= 200`, `s` and `t` contain only lowercase letters and `'#'`.

Example: `s = "ab#c", t = "ad#c"` → `true` (both become `"ac"`). Example: `s = "ab##", t = "c#d#"` → `true` (both become `""`). Example: `s = "a#c", t = "b"` → `false` (`"c"` vs `"b"`).

hint: Building the final strings is easy but costs extra space — a backspace only affects characters to its left, which hints at scanning from the right.
hint: Walk each string from the end; a `'#'` means the next real character to its left is deleted, so count pending skips.
hint: Advance each pointer past its skipped characters to land on the next surviving character, then compare the two survivors; mismatch (or one string running out early) means false.

```cpp
// starter
#include <string>
bool backspaceCompare(std::string s, std::string t);
```

```cpp
bool backspaceCompare(std::string s, std::string t) {
    int i = (int)s.size() - 1, j = (int)t.size() - 1;
    int skipS = 0, skipT = 0;
    while (i >= 0 || j >= 0) {
        while (i >= 0) {
            if (s[i] == '#') { ++skipS; --i; }
            else if (skipS > 0) { --skipS; --i; }
            else break;
        }
        while (j >= 0) {
            if (t[j] == '#') { ++skipT; --j; }
            else if (skipT > 0) { --skipT; --j; }
            else break;
        }
        if (i >= 0 && j >= 0) {
            if (s[i] != t[j]) return false;
        } else if (i >= 0 || j >= 0) {
            return false;
        }
        --i;
        --j;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (backspaceCompare("ab#c", "ad#c") != true) { std::puts("case1"); return 1; }
    if (backspaceCompare("ab##", "c#d#") != true) { std::puts("case2"); return 1; }
    if (backspaceCompare("a#c", "b") != false) { std::puts("case3"); return 1; }
    if (backspaceCompare("a##c", "#a#c") != true) { std::puts("case4"); return 1; }
    if (backspaceCompare("bxj##tw", "bxo#j##tw") != true) { std::puts("case5"); return 1; }
    if (backspaceCompare("y#fo##f", "y#f#o##f") != true) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Because a backspace only erases characters to its left, scanning from the right lets you resolve deletions on the fly without materializing the edited strings. For each string keep a counter of pending skips: a `'#'` increments the skip count, and a normal character is either consumed by a pending skip or is the next surviving character. Advance both pointers to their next survivors and compare; if one string still has a survivor while the other is exhausted, they differ. This runs in O(n + m) time and O(1) extra space, improving on the stack-based O(n + m) space solution.
