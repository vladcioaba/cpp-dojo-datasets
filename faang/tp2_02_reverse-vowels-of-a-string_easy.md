## challenge: Reverse Vowels of a String
tags: two-pointers, string
track: faang
difficulty: easy

Given a string `s`, reverse only the vowels in it and return the resulting string. The vowels are `a`, `e`, `i`, `o`, and `u`, and they can appear in either lowercase or uppercase. All non-vowel characters stay exactly where they are.

Constraints: `1 <= s.length <= 3*10^5`, `s` consists of printable ASCII characters.

Example: `s = "hello"` → `"holle"`. Example: `s = "leetcode"` → `"leotcede"`.

hint: Only the vowels move, and they move to mirror-image positions among themselves — a reversal restricted to a subset.
hint: Scan with two pointers from both ends, each skipping forward past any non-vowel it lands on.
hint: When both pointers rest on a vowel, swap them and step inward; stop once the pointers meet or cross.

```cpp
// starter
#include <string>
std::string reverseVowels(std::string s);
```

```cpp
std::string reverseVowels(std::string s) {
    auto isVowel = [](char c) {
        return c=='a'||c=='e'||c=='i'||c=='o'||c=='u'||
               c=='A'||c=='E'||c=='I'||c=='O'||c=='U';
    };
    int lo = 0, hi = (int)s.size() - 1;
    while (lo < hi) {
        while (lo < hi && !isVowel(s[lo])) ++lo;
        while (lo < hi && !isVowel(s[hi])) --hi;
        if (lo < hi) { std::swap(s[lo], s[hi]); ++lo; --hi; }
    }
    return s;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <utility>
using std::string;
//__USER__
int main() {
    if (reverseVowels("hello") != "holle") { std::puts("case1"); return 1; }
    if (reverseVowels("leetcode") != "leotcede") { std::puts("case2"); return 1; }
    if (reverseVowels("aA") != "Aa") { std::puts("case3"); return 1; }
    if (reverseVowels("bcd") != "bcd") { std::puts("case4"); return 1; }
    if (reverseVowels("Programming") != "Prigrammong") { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Treat the vowels as a sub-sequence that must be reversed while everything else stays fixed. A left pointer and a right pointer walk toward each other; each advances past consonants until it finds a vowel. With both parked on vowels, swap them and continue inward. Because every character is visited at most once by a pointer, the pass is O(n) time and O(1) extra space beyond the returned copy.
