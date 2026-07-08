## challenge: Simplify Path
tags: stack, string
track: faang
difficulty: medium

Given an absolute Unix-style path `path`, return its simplified canonical form. In a Unix path, `.` refers to the current directory, `..` moves up one directory, and multiple consecutive slashes are treated as a single slash. The canonical path must start with a single `/`, join directory names with a single `/`, have no trailing `/` (unless it is the root), and contain no `.` or `..` components. Going up from the root stays at the root.

Constraints: `1 <= path.length <= 3000`; `path` consists of English letters, digits, `.`, `/`, and `_`; `path` is a valid absolute Unix path that begins with `/`.

Example: `path = "/home/"` → `"/home"`. Example: `path = "/../"` → `"/"`. Example: `path = "/home//foo/"` → `"/home/foo"`. Example: `path = "/a/./b/../../c/"` → `"/c"`.

hint: Split the path on `/`; the pieces are directory names plus the special tokens `""`, `"."`, and `".."`.
hint: Walk the pieces left to right keeping a stack of directory names you are currently inside.
hint: Ignore empty pieces and `.`; on `..` pop one directory (if any); otherwise push the name. Join the stack with `/` at the end.

```cpp
// starter
#include <string>
std::string simplifyPath(std::string path);
```

```cpp
std::string simplifyPath(std::string path) {
    std::vector<std::string> st;
    std::string comp;
    path.push_back('/');                 // sentinel so the last component is flushed
    for (char c : path) {
        if (c == '/') {
            if (comp.empty() || comp == ".") {
                // skip
            } else if (comp == "..") {
                if (!st.empty()) st.pop_back();
            } else {
                st.push_back(comp);
            }
            comp.clear();
        } else {
            comp.push_back(c);
        }
    }
    std::string res = "/";
    for (size_t i = 0; i < st.size(); ++i) {
        res += st[i];
        if (i + 1 < st.size()) res += "/";
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
using std::string;
using std::vector;
//__USER__
int main() {
    { if (simplifyPath("/home/")          != "/home")     { std::puts("case1"); return 1; } }
    { if (simplifyPath("/../")            != "/")         { std::puts("case2"); return 1; } }
    { if (simplifyPath("/home//foo/")     != "/home/foo") { std::puts("case3"); return 1; } }
    { if (simplifyPath("/a/./b/../../c/") != "/c")        { std::puts("case4"); return 1; } }
    { if (simplifyPath("/")               != "/")         { std::puts("case5"); return 1; } }
    { if (simplifyPath("/a//b////c/d//././/..") != "/a/b/c") { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Tokenize the path on `/` and process components with a stack. Empty components (from repeated slashes) and `.` are ignored; `..` pops the top directory if the stack is non-empty; any other component is pushed. Appending a trailing `/` sentinel guarantees the final component is emitted by the loop. Joining the surviving stack with single slashes, prefixed by `/`, gives the canonical path in O(n) time and O(n) space.
