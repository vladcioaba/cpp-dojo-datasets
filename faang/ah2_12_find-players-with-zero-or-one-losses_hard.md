## challenge: Find Players With Zero or One Losses
tags: array, hash-table, sorting, counting, arrays-hashing
track: faang
difficulty: hard

You are given an integer array `matches` where `matches[i] = [winner, loser]` indicates that player `winner` defeated player `loser`. Return a list `answer` of size 2 where `answer[0]` is the list of players that have not lost any matches, and `answer[1]` is the list of players that have lost exactly one match. Both lists must be sorted in increasing order, and you should only include players that have played at least one match.

Constraints: `1 <= matches.length <= 10^5`, `matches[i].length == 2`, `1 <= winner, loser <= 10^5`, `winner != loser`, each pair `[winner, loser]` appears at most once.

Example: `matches = [[1,3],[2,3],[3,6],[5,6],[5,7],[4,5],[4,8],[4,9],[10,4],[10,9]]` → `[[1,2,10],[4,5,7,8]]`. Example: `matches = [[2,3],[1,3],[5,4],[6,4]]` → `[[1,2,5,6],[]]`.

hint: Track a loss count for every player who appears anywhere in the matches, including players who only ever win.
hint: Ensure a winner is registered with zero losses even if they never lose, so they can qualify for the zero-loss list.
hint: An ordered map keyed by player id yields both output lists already sorted; collect players with a loss count of 0 and of exactly 1.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> findWinners(std::vector<std::vector<int>>& matches);
```

```cpp
std::vector<std::vector<int>> findWinners(std::vector<std::vector<int>>& matches) {
    std::map<int, int> losses;
    for (auto& m : matches) {
        if (!losses.count(m[0])) losses[m[0]] = 0;
        losses[m[1]]++;
    }
    std::vector<int> zero, one;
    for (auto& [player, l] : losses) {
        if (l == 0) zero.push_back(player);
        else if (l == 1) one.push_back(player);
    }
    return {zero, one};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <map>
using std::vector;
static bool eq(const vector<int>& a, const vector<int>& b) { return a == b; }
//__USER__
int main() {
    {
        vector<vector<int>> m{{1,3},{2,3},{3,6},{5,6},{5,7},{4,5},{4,8},{4,9},{10,4},{10,9}};
        auto r = findWinners(m);
        if (!(r.size()==2 && eq(r[0], {1,2,10}) && eq(r[1], {4,5,7,8}))) { std::puts("case1"); return 1; }
    }
    {
        vector<vector<int>> m{{2,3},{1,3},{5,4},{6,4}};
        auto r = findWinners(m);
        if (!(r.size()==2 && eq(r[0], {1,2,5,6}) && eq(r[1], {}))) { std::puts("case2"); return 1; }
    }
    {
        vector<vector<int>> m{{1,2}};
        auto r = findWinners(m);
        if (!(r.size()==2 && eq(r[0], {1}) && eq(r[1], {2}))) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Give every player who appears in any match a loss counter. Winners are inserted with zero losses so they remain candidates for the no-loss list, while each match increments the loser's count. Iterating an ordered map (`std::map`) keyed by player id naturally produces both answer lists in ascending order in a single sweep: zero-loss players and exactly-one-loss players. O(m log p) time for `m` matches and `p` distinct players.
