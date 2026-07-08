## challenge: Reconstruct Itinerary
tags: graph, eulerian-path, dfs, hierholzer
track: faang
difficulty: hard

You are given a list of airline `tickets` where `tickets[i] = [from, to]` represents the departure and arrival airports of one flight. Reconstruct the itinerary in order and return it. All of the tickets belong to a person who departs from `"JFK"`, so the itinerary must begin with `"JFK"`. If there are multiple valid itineraries, return the one that is lexicographically smallest when read as a single list of airport codes. You must use all the tickets exactly once each; the input guarantees at least one valid itinerary exists.

Constraints: `1 <= tickets.length <= 300`, `tickets[i].length == 2`, airport codes are 3 uppercase letters, `from != to`.

Example: `tickets = [["MUC","LHR"],["JFK","MUC"],["SFO","SJC"],["LHR","SFO"]]` → `["JFK","MUC","LHR","SFO","SJC"]`. Example: `tickets = [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]` → `["JFK","ATL","JFK","SFO","ATL","SFO"]`.

hint: This is an Eulerian path problem: use every edge exactly once. To get the lexicographically smallest result, always take the smallest available destination first.
hint: Hierholzer's algorithm — store destinations per airport in a sorted multiset, DFS greedily consuming edges, and prepend each airport to the route as its recursion unwinds (append then reverse).

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> findItinerary(std::vector<std::vector<std::string>>& tickets);
```

```cpp
#include <unordered_map>
#include <set>
#include <functional>
#include <algorithm>
std::vector<std::string> findItinerary(std::vector<std::vector<std::string>>& tickets) {
    std::unordered_map<std::string, std::multiset<std::string>> g;
    for (auto& t : tickets) g[t[0]].insert(t[1]);
    std::vector<std::string> route;
    std::function<void(const std::string&)> dfs = [&](const std::string& u) {
        auto& dests = g[u];
        while (!dests.empty()) {
            std::string v = *dests.begin();
            dests.erase(dests.begin());
            dfs(v);
        }
        route.push_back(u);
    };
    dfs("JFK");
    std::reverse(route.begin(), route.end());
    return route;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
using std::vector;
using std::string;
//__USER__
int main() {
    {
        vector<vector<string>> t{{"MUC","LHR"},{"JFK","MUC"},{"SFO","SJC"},{"LHR","SFO"}};
        auto r = findItinerary(t);
        vector<string> e{"JFK","MUC","LHR","SFO","SJC"};
        if (r != e) { std::puts("case1"); return 1; }
    }
    {
        vector<vector<string>> t{{"JFK","SFO"},{"JFK","ATL"},{"SFO","ATL"},{"ATL","JFK"},{"ATL","SFO"}};
        auto r = findItinerary(t);
        vector<string> e{"JFK","ATL","JFK","SFO","ATL","SFO"};
        if (r != e) { std::puts("case2"); return 1; }
    }
    {
        vector<vector<string>> t{{"JFK","KUL"},{"JFK","NRT"},{"NRT","JFK"}};
        auto r = findItinerary(t);
        vector<string> e{"JFK","NRT","JFK","KUL"};
        if (r != e) { std::puts("case3"); return 1; }
    }
    {
        vector<vector<string>> t{{"JFK","A"},{"A","JFK"}};
        auto r = findItinerary(t);
        vector<string> e{"JFK","A","JFK"};
        if (r != e) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Using every ticket exactly once is an Eulerian path starting at `"JFK"`. Hierholzer's algorithm builds it: keep each airport's outgoing destinations in a sorted multiset so the smallest is always chosen first, guaranteeing the lexicographically smallest itinerary. DFS greedily consumes edges; when an airport has no remaining outgoing tickets, append it to the route. The route is built in reverse (a dead-end airport is added first), so reverse it at the end. Time is O(E log E) for the multisets, space O(E).
