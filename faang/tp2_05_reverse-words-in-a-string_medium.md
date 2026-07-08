## challenge: Reverse Words in a String
tags: two-pointers, string
track: faang
difficulty: medium

Given a string `s` containing words separated by spaces, return a string with the words in reverse order. A word is a maximal run of non-space characters. Collapse any runs of multiple spaces into a single space in the output, and remove all leading and trailing spaces.

Constraints: `1 <= s.length <= 10^4`, `s` contains English letters, digits, and spaces, and there is at least one word.

Example: `s = "the sky is blue"` → `"blue is sky the"`. Example: `s = "  hello world  "` → `"world hello"`. Example: `s = "a good   example"` → `"example good a"`.

hint: The real work is isolating words while skipping arbitrary stretches of whitespace between and around them.
hint: Use two indices to bracket each word: advance past spaces to find its start, then advance to the first following space to find its end.
hint: Collect the words in order, then emit them back to front joined by exactly one space.

```cpp
// starter
#include <string>
std::string reverseWords(std::string s);
```

```cpp
std::string reverseWords(std::string s) {
    std::vector<std::string> words;
    int i = 0, n = (int)s.size();
    while (i < n) {
        while (i < n && s[i] == ' ') ++i;
        if (i >= n) break;
        int j = i;
        while (j < n && s[j] != ' ') ++j;
        words.push_back(s.substr(i, j - i));
        i = j;
    }
    std::string res;
    for (int k = (int)words.size() - 1; k >= 0; --k) {
        res += words[k];
        if (k > 0) res += ' ';
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
//__USER__
int main() {
    if (reverseWords("the sky is blue") != "blue is sky the") { std::puts("case1"); return 1; }
    if (reverseWords("  hello world  ") != "world hello") { std::puts("case2"); return 1; }
    if (reverseWords("a good   example") != "example good a") { std::puts("case3"); return 1; }
    if (reverseWords("single") != "single") { std::puts("case4"); return 1; }
    if (reverseWords("  leading") != "leading") { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Tokenize with two indices: skip any spaces to reach a word's start `i`, then extend `j` to the space (or end) that terminates it, and record `s[i..j)`. Repeating this cleanly handles leading, trailing, and repeated internal spaces. Finally rebuild the answer by concatenating the collected words in reverse order, inserting one space between consecutive words. O(n) time and O(n) space.
