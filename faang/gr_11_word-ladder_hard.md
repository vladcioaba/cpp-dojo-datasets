## challenge: Word Ladder

tags: graph, bfs, hash-table

track: faang
difficulty: hard

Given two words `beginWord` and `endWord` and a dictionary `wordList`, a transformation sequence changes one letter at a time such that every intermediate word (and the final word) is in `wordList`. Return the number of words in the shortest transformation sequence from `beginWord` to `endWord`, counting both endpoints, or `0` if no such sequence exists. `beginWord` need not be in `wordList`; all words have the same length and consist of lowercase letters.

Constraints: `1 <= word length <= 10`, `1 <= wordList.length <= 5000`, all words are the same length and lowercase, `endWord` may or may not be present, words in `wordList` are unique.

Example: `beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]` → `5` (`hit → hot → dot → dog → cog`). Example: same words but `wordList = ["hot","dot","dog","lot","log"]` → `0` (`cog` is unreachable).

hint: Each word is a node and two words are adjacent when they differ in exactly one letter; the shortest sequence is a shortest path, so use BFS.
hint: Building all pairwise adjacencies is expensive — instead generate a word's neighbours on the fly by trying each of the `26` letters at each position.
hint: Track the level as you expand rings of the BFS, and erase words from the dictionary the moment you enqueue them so you never revisit a node.

```cpp
// starter
#include <string>
#include <vector>
int ladderLength(std::string beginWord, std::string endWord, std::vector<std::string>& wordList);
```

```cpp
int ladderLength(std::string beginWord, std::string endWord, std::vector<std::string>& wordList) {
    std::unordered_set<std::string> dict(wordList.begin(), wordList.end());
    if (!dict.count(endWord)) return 0;
    std::queue<std::string> q;
    q.push(beginWord);
    int steps = 1;
    while (!q.empty()) {
        int sz = (int)q.size();
        for (int i = 0; i < sz; ++i) {
            std::string w = q.front(); q.pop();
            if (w == endWord) return steps;
            for (int j = 0; j < (int)w.size(); ++j) {
                char orig = w[j];
                for (char c = 'a'; c <= 'z'; ++c) {
                    if (c == orig) continue;
                    w[j] = c;
                    if (dict.count(w)) { q.push(w); dict.erase(w); }
                }
                w[j] = orig;
            }
        }
        ++steps;
    }
    return 0;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <unordered_set>
#include <queue>
using std::vector;
using std::string;
//__USER__
int main() {
    { vector<string> w{"hot","dot","dog","lot","log","cog"}; if (ladderLength("hit", "cog", w) != 5) { std::puts("case1"); return 1; } }
    { vector<string> w{"hot","dot","dog","lot","log"};       if (ladderLength("hit", "cog", w) != 0) { std::puts("case2"); return 1; } }
    { vector<string> w{"a","b","c"};                          if (ladderLength("a", "c", w)   != 2) { std::puts("case3"); return 1; } }
    { vector<string> w{"hot","dog"};                          if (ladderLength("hot", "dog", w) != 0) { std::puts("case4"); return 1; } }
    { vector<string> w{"hot"};                                if (ladderLength("hit", "hot", w) != 2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Model words as graph nodes with an edge between words differing by one letter, then BFS from `beginWord` for the shortest path. Rather than precompute edges, expand a word by substituting each of `26` letters at each position and keeping candidates present in a dictionary set. Process the queue one ring at a time, incrementing the length per ring, and erase each enqueued word to avoid revisits. With `L` = word length and `W` = dictionary size, that is O(W * L * 26) time, O(W * L) space.
