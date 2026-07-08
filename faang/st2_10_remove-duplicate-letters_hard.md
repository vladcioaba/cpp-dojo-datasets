## challenge: Remove Duplicate Letters
tags: monotonic-stack, stack, greedy, string
track: faang
difficulty: hard

Given a string `s`, remove duplicate letters so that every letter appears exactly once. Among all strings that can be formed this way, return the one that is the smallest in lexicographical order.

Constraints: `1 <= s.length <= 10^4`; `s` consists of lowercase English letters.

Example: `s = "bcabc"` → `"abc"`. Example: `s = "cbacdcbc"` → `"acdb"`.

hint: Build the answer greedily on a stack, keeping the kept letters as increasing as possible.
hint: Before pushing a new letter, you may pop a larger letter on top — but only if that letter appears again later, so it is safe to drop now.
hint: Track each letter's last occurrence index and whether it is already on the stack to avoid duplicates.

```cpp
// starter
#include <string>
std::string removeDuplicateLetters(std::string s);
```

```cpp
std::string removeDuplicateLetters(std::string s) {
    int last[26] = {0};
    for (int i = 0; i < (int)s.size(); ++i) last[s[i] - 'a'] = i;
    bool inStack[26] = {false};
    std::string st;
    for (int i = 0; i < (int)s.size(); ++i) {
        char c = s[i];
        if (inStack[c - 'a']) continue;                 // already placed
        while (!st.empty() && st.back() > c && last[st.back() - 'a'] > i) {
            inStack[st.back() - 'a'] = false;           // safe to drop: it recurs later
            st.pop_back();
        }
        st.push_back(c);
        inStack[c - 'a'] = true;
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
    if (removeDuplicateLetters("bcabc")    != "abc")  { std::puts("case1"); return 1; }
    if (removeDuplicateLetters("cbacdcbc") != "acdb") { std::puts("case2"); return 1; }
    if (removeDuplicateLetters("abcd")     != "abcd") { std::puts("case3"); return 1; }
    if (removeDuplicateLetters("aaaa")     != "a")    { std::puts("case4"); return 1; }
    if (removeDuplicateLetters("bbcaac")   != "bac")  { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Greedily construct the result on a stack whose letters stay in the smallest achievable increasing arrangement. First record each letter's last index. Scanning left to right, skip letters already on the stack. Otherwise, while the top is larger than the current letter and that top letter still occurs later in the string, pop it — deferring it is safe and yields a lexicographically smaller prefix. Then push the current letter and mark it present. Because a letter is popped only when guaranteed to reappear, every distinct letter ends up exactly once, in the optimal order. Each character is pushed and popped at most once, so the algorithm is O(n) time and O(1) extra space (fixed 26-letter arrays).
