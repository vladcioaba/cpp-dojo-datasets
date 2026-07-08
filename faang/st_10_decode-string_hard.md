## challenge: Decode String
tags: stack, string
track: faang
difficulty: hard

Given an encoded string, return its decoded form. The encoding rule is `k[encoded_string]`, meaning the `encoded_string` inside the brackets is repeated exactly `k` times. `k` is a positive integer and the brackets may be nested arbitrarily. The input is always well-formed: brackets are balanced, and digits appear only as repetition counts immediately before a `[`, never inside the decoded content. The decoded output length fits comfortably in memory.

Constraints: `1 <= s.length <= 30`; `s` consists of lowercase English letters, digits, and the characters `[` and `]`; every repetition count `k` satisfies `1 <= k <= 300`; the fully decoded string has length at most `10^5`.

Example: `s = "3[a]2[bc]"` → `"aaabcbc"`. Example: `s = "3[a2[c]]"` → `"accaccacc"`. Example: `s = "2[abc]3[cd]ef"` → `"abcabccdcdcdef"`.

hint: When you hit `[`, you must pause the string you were building and start a fresh one for the bracket's contents.
hint: On `]`, you finish the inner string, repeat it `k` times, and append it to the string you paused — a naturally nested, stack-like process.
hint: Use two stacks: one for the pending repetition counts and one for the partially built strings at each nesting level.

```cpp
// starter
#include <string>
std::string decodeString(std::string s);
```

```cpp
std::string decodeString(std::string s) {
    std::vector<int> counts;         // repetition counts, one per open bracket
    std::vector<std::string> parts;  // string built before each open bracket
    std::string cur;
    int k = 0;
    for (char c : s) {
        if (c >= '0' && c <= '9') {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            counts.push_back(k);
            parts.push_back(cur);
            k = 0;
            cur.clear();
        } else if (c == ']') {
            int rep = counts.back(); counts.pop_back();
            std::string prev = parts.back(); parts.pop_back();
            std::string repeated;
            for (int i = 0; i < rep; ++i) repeated += cur;
            cur = prev + repeated;
        } else {
            cur.push_back(c);
        }
    }
    return cur;
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
    { if (decodeString("3[a]2[bc]")     != "aaabcbc")            { std::puts("case1"); return 1; } }
    { if (decodeString("3[a2[c]]")      != "accaccacc")          { std::puts("case2"); return 1; } }
    { if (decodeString("2[abc]3[cd]ef") != "abcabccdcdcdef")     { std::puts("case3"); return 1; } }
    { if (decodeString("abc")           != "abc")                { std::puts("case4"); return 1; } }
    { if (decodeString("10[a]")         != "aaaaaaaaaa")         { std::puts("case5"); return 1; } }
    { if (decodeString("2[a2[b]c]")     != "abbcabbc")           { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Scan the string maintaining the current level's partial result `cur` and the running number `k`. Digits accumulate into `k`. An opening bracket starts a new level: push `k` and the outer `cur` onto two stacks, then reset both. A closing bracket finishes the level: pop the repetition count and the outer prefix, repeat `cur` that many times, and append it to the prefix to become the new `cur`. Letters simply extend `cur`. The stacks capture arbitrary nesting; the work is linear in the total decoded length.
