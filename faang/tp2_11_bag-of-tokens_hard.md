## challenge: Bag of Tokens
tags: two-pointers, array, sorting, greedy
track: faang
difficulty: hard

You start with an integer `power` and a score of `0`, and you hold `tokens` where `tokens[i]` is the value of the i-th token. Each token may be played at most once, in either direction: play it **face up** — if `power >= tokens[i]`, lose `tokens[i]` power and gain `1` score; or play it **face down** — if `score >= 1`, lose `1` score and gain `tokens[i]` power. Return the maximum score achievable.

Constraints: `0 <= tokens.length <= 1000`, `0 <= tokens[i], power < 10^4`.

Example: `tokens = [100], power = 50` → `0`. Example: `tokens = [100,200], power = 150` → `1`. Example: `tokens = [100,200,300,400], power = 200` → `2`.

hint: Sort the tokens: to gain score cheaply you want to buy the smallest tokens, and to buy back power you want to sell the largest.
hint: Run two pointers inward — spend power on the cheapest token when you can afford it, otherwise trade one score for the most valuable remaining token.
hint: Track the best score seen, since a face-down move temporarily lowers your score but may unlock further gains.

```cpp
// starter
#include <vector>
int bagOfTokensScore(std::vector<int>& tokens, int power);
```

```cpp
int bagOfTokensScore(std::vector<int>& tokens, int power) {
    std::sort(tokens.begin(), tokens.end());
    int lo = 0, hi = (int)tokens.size() - 1;
    int score = 0, best = 0;
    while (lo <= hi) {
        if (power >= tokens[lo]) {
            power -= tokens[lo];
            ++score; ++lo;
            best = std::max(best, score);
        } else if (score > 0 && lo < hi) {
            power += tokens[hi];
            --score; --hi;
        } else {
            break;
        }
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> t{100}; if (bagOfTokensScore(t, 50) != 0) { std::puts("case1"); return 1; } }
    { vector<int> t{100,200}; if (bagOfTokensScore(t, 150) != 1) { std::puts("case2"); return 1; } }
    { vector<int> t{100,200,300,400}; if (bagOfTokensScore(t, 200) != 2) { std::puts("case3"); return 1; } }
    { vector<int> t{}; if (bagOfTokensScore(t, 100) != 0) { std::puts("case4"); return 1; } }
    { vector<int> t{71,55,82}; if (bagOfTokensScore(t, 54) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort the tokens so the cheapest sit at the front and the richest at the back. Greedily buy score with the smallest affordable token (`power -= tokens[lo]`, `++score`); when you cannot afford the current cheapest but have score to spare, sell your most valuable token face down (`power += tokens[hi]`, `--score`) to refuel. The pointers close in, and because a face-down trade dips your score before it can rise again, record the best score ever held. Require `lo < hi` before trading so you never sell the very token you are trying to buy. O(n log n) time, O(1) space.
