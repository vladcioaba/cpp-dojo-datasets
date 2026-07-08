## challenge: Long Pressed Name
tags: two-pointers, string
track: faang
difficulty: medium

Your friend types their `name` into a keyboard, but sometimes a key gets long-pressed and a character is registered one or more extra times. Given the intended `name` and the actually-typed string `typed`, return `true` if `typed` could have resulted from typing `name` with some characters possibly long-pressed.

Constraints: `1 <= name.length, typed.length <= 1000`, both strings consist of lowercase English letters.

Example: `name = "alex", typed = "aaleex"` → `true`. Example: `name = "saeed", typed = "ssaaedd"` → `false` (the second `e` in `name` is never typed).

hint: Walk both strings with two pointers, matching each character of `name` against `typed`.
hint: When the current characters differ, the only legal explanation is that `typed` repeated its previous character — a long press.
hint: If a `typed` character neither matches `name` nor repeats the prior one, it fails; at the end every character of `name` must have been consumed.

```cpp
// starter
#include <string>
bool isLongPressedName(std::string name, std::string typed);
```

```cpp
bool isLongPressedName(std::string name, std::string typed) {
    int i = 0, j = 0;
    int n = (int)name.size(), m = (int)typed.size();
    while (j < m) {
        if (i < n && name[i] == typed[j]) { ++i; ++j; }
        else if (j > 0 && typed[j] == typed[j - 1]) { ++j; }
        else return false;
    }
    return i == n;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
using std::string;
//__USER__
int main() {
    if (isLongPressedName("alex", "aaleex") != true) { std::puts("case1"); return 1; }
    if (isLongPressedName("saeed", "ssaaedd") != false) { std::puts("case2"); return 1; }
    if (isLongPressedName("leelee", "lleeelee") != true) { std::puts("case3"); return 1; }
    if (isLongPressedName("laiden", "laiden") != true) { std::puts("case4"); return 1; }
    if (isLongPressedName("alex", "aaleexa") != false) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Advance a pointer `i` through `name` and `j` through `typed`. When the characters match, consume both. When they differ, the only way `typed` can be valid is if `typed[j]` repeats `typed[j-1]` — an extra registration from a long press — in which case skip it. Any other mismatch is fatal. After exhausting `typed`, success requires that all of `name` was matched (`i == n`). Each pointer only moves forward, giving O(n + m) time and O(1) space.
