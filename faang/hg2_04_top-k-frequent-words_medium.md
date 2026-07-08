## challenge: Top K Frequent Words
tags: heap, priority-queue, hash-table, sorting

track: faang
difficulty: medium

Given an array of strings `words` and an integer `k`, return the `k` most frequent words. Sort the result by descending frequency; words with the same frequency are ordered lexicographically (ascending).

Constraints: `1 <= words.length <= 500`, `1 <= words[i].length <= 10`, `words[i]` is lowercase, `1 <= k <= number of distinct words`.

Example: `words = ["i","love","leetcode","i","love","coding"], k = 2` → `["i","love"]` (both appear twice; `"i"` precedes `"love"`). Example: `words = ["the","day","is","sunny","the","the","the","sunny","is","is"], k = 4` → `["the","is","sunny","day"]`.

hint: Count each word, then you need the `k` best under a two-level ordering: frequency first, then lexicographic tie-break.
hint: A size-`k` heap works if its top is the *worst* remaining candidate, so pushing beyond `k` pops that worst one.
hint: Make "better" mean higher frequency or, on a tie, a lexicographically smaller word; the heap keeps the best `k`, and you read them out in reverse.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> topKFrequent(std::vector<std::string>& words, int k);
```

```cpp
std::vector<std::string> topKFrequent(std::vector<std::string>& words, int k) {
    std::unordered_map<std::string, int> cnt;
    for (auto& w : words) cnt[w]++;
    // Min-heap whose top is the worst candidate: lower freq is worse,
    // and on equal freq a lexicographically larger word is worse.
    auto cmp = [](const std::pair<int, std::string>& a, const std::pair<int, std::string>& b) {
        if (a.first != b.first) return a.first > b.first;   // higher freq = better = "smaller"
        return a.second < b.second;                          // smaller word = better = "smaller"
    };
    std::priority_queue<std::pair<int, std::string>,
                        std::vector<std::pair<int, std::string>>, decltype(cmp)> pq(cmp);
    for (auto& [w, c] : cnt) {
        pq.push({c, w});
        if ((int)pq.size() > k) pq.pop();
    }
    std::vector<std::string> res(k);
    for (int i = k - 1; i >= 0; --i) { res[i] = pq.top().second; pq.pop(); }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <queue>
#include <utility>
#include <unordered_map>
using std::vector;
using std::string;
//__USER__
int main() {
    { vector<string> w{"i","love","leetcode","i","love","coding"};
      if (topKFrequent(w, 2) != vector<string>({"i","love"})) { std::puts("case1"); return 1; } }
    { vector<string> w{"the","day","is","sunny","the","the","the","sunny","is","is"};
      if (topKFrequent(w, 4) != vector<string>({"the","is","sunny","day"})) { std::puts("case2"); return 1; } }
    { vector<string> w{"a","aa","aaa"};
      if (topKFrequent(w, 1) != vector<string>({"a"})) { std::puts("case3"); return 1; } }
    { vector<string> w{"b","a"};
      if (topKFrequent(w, 2) != vector<string>({"a","b"})) { std::puts("case4"); return 1; } }
    { vector<string> w{"aaa","bbb","aaa","ccc","bbb","aaa"};
      if (topKFrequent(w, 2) != vector<string>({"aaa","bbb"})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** After counting occurrences, the goal is the `k` best pairs under a compound order — frequency descending, word ascending on ties. Maintain a bounded min-heap whose comparator makes "better" pairs compare as smaller so the heap's top is always the current worst; whenever it grows past `k`, pop that worst. Emptying the heap yields the survivors worst-first, so fill the output back-to-front. Building costs O(n) plus O(u log k) heap work for `u` distinct words, O(u) space.
