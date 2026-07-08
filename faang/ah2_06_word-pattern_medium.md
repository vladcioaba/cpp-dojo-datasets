## challenge: Word Pattern
tags: string, hash-table, arrays-hashing
track: faang
difficulty: medium

Given a `pattern` and a string `s`, find if `s` follows the same pattern. Here "follow" means a full bijection between letters in `pattern` and non-empty words in `s`: each letter maps to exactly one word and each word maps to exactly one letter, preserving order. Words in `s` are separated by single spaces.

Constraints: `1 <= pattern.length <= 300`, `pattern` contains only lowercase English letters. `1 <= s.length <= 3000`, `s` contains lowercase English letters and single spaces, with no leading or trailing spaces.

Example: `pattern = "abba", s = "dog cat cat dog"` → `true`. Example: `pattern = "abba", s = "dog cat cat fish"` → `false`. Example: `pattern = "aaaa", s = "dog cat cat dog"` → `false`. Example: `pattern = "abba", s = "dog dog dog dog"` → `false`.

hint: First split `s` into words; if the number of words differs from the pattern length, it cannot follow.
hint: This is a two-way mapping problem, just like isomorphic strings but with letters on one side and whole words on the other.
hint: Maintain a map from letter to word and a map from word to letter, and reject any position where either map already binds its key to a different partner.

```cpp
// starter
#include <string>
bool wordPattern(std::string pattern, std::string s);
```

```cpp
bool wordPattern(std::string pattern, std::string s) {
    std::vector<std::string> words;
    std::string cur;
    std::istringstream iss(s);
    while (iss >> cur) words.push_back(cur);
    if (words.size() != pattern.size()) return false;
    std::unordered_map<char, std::string> p2w;
    std::unordered_map<std::string, char> w2p;
    for (size_t i = 0; i < words.size(); ++i) {
        char c = pattern[i];
        const std::string& w = words[i];
        if (p2w.count(c) && p2w[c] != w) return false;
        if (w2p.count(w) && w2p[w] != c) return false;
        p2w[c] = w;
        w2p[w] = c;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <sstream>
#include <unordered_map>
using std::string;
//__USER__
int main() {
    if (wordPattern("abba", "dog cat cat dog") != true) { std::puts("case1"); return 1; }
    if (wordPattern("abba", "dog cat cat fish") != false) { std::puts("case2"); return 1; }
    if (wordPattern("aaaa", "dog cat cat dog") != false) { std::puts("case3"); return 1; }
    if (wordPattern("abba", "dog dog dog dog") != false) { std::puts("case4"); return 1; }
    if (wordPattern("abc", "b c a") != true) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Tokenize `s` into words and immediately reject a length mismatch with `pattern`. Then it reduces to checking a bijection between letters and words, mirrored by two hash maps (letter→word and word→letter). Each aligned pair is legal only if neither map already ties its key to a different partner, which blocks both the "one letter, two words" and "two letters, one word" failures. Linear in the input size.
