## challenge: Maximum Number of Vowels in a Substring of Given Length
tags: string, sliding-window
track: faang
difficulty: easy

Given a string `s` and an integer `k`, return the maximum number of vowels (`a`, `e`, `i`, `o`, `u`) contained in any substring of `s` with length exactly `k`.

Constraints: `1 <= k <= s.length <= 10^5`, `s` consists of lowercase English letters.

Example: `s = "abciiidef", k = 3` → `3` (the substring `"iii"`). Example: `s = "aeiou", k = 2` → `2`. Example: `s = "leetcode", k = 3` → `2` (`"eet"`).

hint: Fixed-length windows again — you never need to recount an entire window from scratch.
hint: Keep a running vowel count; when the window advances, add 1 if the entering char is a vowel and subtract 1 if the leaving char was.
hint: The answer can never exceed `k`, so tracking the running maximum as you slide is all you need.

```cpp
// starter
#include <string>
int maxVowels(std::string s, int k);
```

```cpp
int maxVowels(std::string s, int k) {
    auto isVowel = [](char c) {
        return c=='a'||c=='e'||c=='i'||c=='o'||c=='u';
    };
    int cur = 0;
    for (int i = 0; i < k; ++i) if (isVowel(s[i])) ++cur;
    int best = cur;
    for (int i = k; i < (int)s.size(); ++i) {
        if (isVowel(s[i])) ++cur;
        if (isVowel(s[i - k])) --cur;
        best = std::max(best, cur);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <algorithm>
using std::string;
//__USER__
int main() {
    if (maxVowels("abciiidef",3) != 3) { std::puts("case1"); return 1; }
    if (maxVowels("aeiou",2) != 2) { std::puts("case2"); return 1; }
    if (maxVowels("leetcode",3) != 2) { std::puts("case3"); return 1; }
    if (maxVowels("rhythms",4) != 0) { std::puts("case4"); return 1; }
    if (maxVowels("tryhard",1) != 1) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Count the vowels in the first window of length `k`, then slide the window one position at a time. Each step is O(1): increment the count if the new right-hand character is a vowel, decrement it if the character leaving on the left was a vowel. Track the running maximum. O(n) time, O(1) space — no need to rescan the window at each position.
