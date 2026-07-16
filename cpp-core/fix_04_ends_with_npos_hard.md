## challenge: fix: the suffix check that says yes to longer suffixes
tags: code-review, debugging, strings
track: core
difficulty: hard

This code review found a bug: `endsWith("bc", "abc")` returns true — a string is reported as ending with a suffix that is longer than the string itself. Find and fix it — keep the function signature.

hint: What does rfind return when the suffix is not found at all, and what does the right-hand side compute when the suffix is longer than the string?
hint: This is unsigned wraparound colliding with a sentinel value.
hint: s.size() - suffix.size() wraps to SIZE_MAX when the suffix is one longer than s — and SIZE_MAX is exactly std::string::npos, the same value rfind returns for "not found", so the two compare equal.

```cpp
// starter
bool endsWith(const std::string& s, const std::string& suffix) {
    return s.rfind(suffix) == s.size() - suffix.size();
}
```

```cpp
bool endsWith(const std::string& s, const std::string& suffix) {
    if (suffix.size() > s.size()) {
        return false;
    }
    return s.compare(s.size() - suffix.size(), suffix.size(), suffix) == 0;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(endsWith("catalog", "log"));
    assert(!endsWith("catalog", "dog"));
    assert(endsWith("abc", "abc"));
    assert(endsWith("catalog", ""));
    assert(!endsWith("bc", "abc"));    // suffix longer than the string: buggy version says true
    assert(!endsWith("", "x"));
    std::puts("PASS");
}
```

**Editorial:** Both sides of the comparison misbehave in the same corner: when `suffix` is longer than `s`, `rfind` returns `npos` (defined as `size_t(-1)`, i.e. `SIZE_MAX`), and the unsigned subtraction `s.size() - suffix.size()` wraps around — for a suffix exactly one character longer it wraps to `SIZE_MAX` too, so "not found" compares equal to the garbage index and the function answers true. The fix rejects oversized suffixes up front and then compares the tail directly with `compare` (in C++20 you would just call `s.ends_with(suffix)`). Reviewers should treat any arithmetic on `size()` values that can go "negative", and any `==` against a value that might be `npos`, as a wraparound trap.
