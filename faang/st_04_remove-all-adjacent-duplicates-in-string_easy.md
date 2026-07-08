## challenge: Remove All Adjacent Duplicates In String
tags: stack, string
track: faang
difficulty: easy

You are given a string `s` of lowercase English letters. A duplicate removal deletes two adjacent equal letters. Repeatedly perform duplicate removals on `s` until no two adjacent letters are equal, and return the final string. The result is guaranteed to be unique.

Constraints: `1 <= s.length <= 10^5`; `s` consists of lowercase English letters.

Example: `s = "abbaca"` → `"ca"` (remove `"bb"` to get `"aaca"`, then `"aa"` to get `"ca"`). Example: `s = "azxxzy"` → `"ay"`.

hint: Deleting a pair can expose a new adjacent pair, so a single left-to-right pass with rescanning would be O(n^2).
hint: Push characters onto a stack; the top of the stack is the character immediately to the left in the result so far.
hint: If the next character equals the stack top, they annihilate — pop instead of pushing.

```cpp
// starter
#include <string>
std::string removeDuplicates(std::string s);
```

```cpp
std::string removeDuplicates(std::string s) {
    std::string st;   // acts as a stack of surviving characters
    for (char c : s) {
        if (!st.empty() && st.back() == c) st.pop_back();
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
    { if (removeDuplicates("abbaca")  != "ca") { std::puts("case1"); return 1; } }
    { if (removeDuplicates("azxxzy")  != "ay") { std::puts("case2"); return 1; } }
    { if (removeDuplicates("aaaaaaaa") != "") { std::puts("case3"); return 1; } }
    { if (removeDuplicates("abcde")   != "abcde") { std::puts("case4"); return 1; } }
    { if (removeDuplicates("aabccba") != "a") { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Build the answer as a stack. Scan each character: if it matches the current top, the two are an adjacent duplicate and cancel, so pop the top; otherwise push the character. Because removals can cascade, the stack naturally handles newly exposed pairs without rescanning. Each character is pushed and popped at most once, giving O(n) time and O(n) space; the surviving stack is the answer.
