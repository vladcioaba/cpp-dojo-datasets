## challenge: Isomorphic Strings
tags: string, hash-table, arrays-hashing
track: faang
difficulty: easy

Two strings `s` and `t` are isomorphic if the characters of `s` can be replaced to get `t`, where every occurrence of a character maps to the same character and no two characters map to the same character (the mapping is a bijection). Order must be preserved. Return `true` if `s` and `t` are isomorphic.

Constraints: `1 <= s.length <= 5*10^4`, `t.length == s.length`, both strings consist of printable ASCII characters.

Example: `s = "egg", t = "add"` → `true` (`e→a`, `g→d`). Example: `s = "foo", t = "bar"` → `false` (`o` would map to both `a` and `r`). Example: `s = "paper", t = "title"` → `true`.

hint: A single map from `s` characters to `t` characters is not enough, since two different `s` characters could collide onto the same `t` character.
hint: Maintain two maps: one from `s` to `t` and one from `t` to `s`, enforcing consistency in both directions.
hint: At each position, if either map already records a different partner for the current character, the strings cannot be isomorphic; otherwise record both directions and continue.

```cpp
// starter
#include <string>
bool isIsomorphic(std::string s, std::string t);
```

```cpp
bool isIsomorphic(std::string s, std::string t) {
    if (s.size() != t.size()) return false;
    std::unordered_map<char, char> s2t, t2s;
    for (size_t i = 0; i < s.size(); ++i) {
        char a = s[i], b = t[i];
        if (s2t.count(a) && s2t[a] != b) return false;
        if (t2s.count(b) && t2s[b] != a) return false;
        s2t[a] = b;
        t2s[b] = a;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <unordered_map>
using std::string;
//__USER__
int main() {
    if (isIsomorphic("egg", "add") != true) { std::puts("case1"); return 1; }
    if (isIsomorphic("foo", "bar") != false) { std::puts("case2"); return 1; }
    if (isIsomorphic("paper", "title") != true) { std::puts("case3"); return 1; }
    if (isIsomorphic("badc", "baba") != false) { std::puts("case4"); return 1; }
    if (isIsomorphic("a", "a") != true) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Isomorphism requires a consistent one-to-one correspondence, so track the mapping in both directions with two hash maps. Walking the strings in lockstep, a character pair is valid only if neither map has already committed the character to a different partner. Any conflict fails immediately; surviving to the end proves the bijection holds. O(n) time, O(1) space over a bounded alphabet.
