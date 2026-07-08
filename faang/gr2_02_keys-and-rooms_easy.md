## challenge: Keys and Rooms
tags: graph, dfs, bfs
track: faang
difficulty: easy

There are `n` rooms labeled from `0` to `n - 1`, and all rooms are locked except room `0`. Your goal is to visit all the rooms. When you visit a room you may pick up all the keys inside it. Each key opens exactly one room. Given `rooms` where `rooms[i]` is the list of keys found in room `i`, return `true` if and only if you can visit every room, starting from room `0`.

Constraints: `n == rooms.length`, `2 <= n <= 1000`, `0 <= rooms[i].length <= 1000`, each key is in `[0, n)`.

Example: `rooms = [[1],[2],[3],[]]` → `true` (0 → 1 → 2 → 3). Example: `rooms = [[1,3],[3,0,1],[2],[0]]` → `false` (room 2 is never opened).

hint: This is graph reachability: rooms are nodes and keys are directed edges. Can you reach every node from node 0?
hint: Run a DFS or BFS from room 0, marking rooms visited as you collect their keys, then check whether every room was reached.

```cpp
// starter
#include <vector>
bool canVisitAllRooms(std::vector<std::vector<int>>& rooms);
```

```cpp
bool canVisitAllRooms(std::vector<std::vector<int>>& rooms) {
    int n = (int)rooms.size();
    std::vector<char> visited(n, 0);
    std::vector<int> stack{0};
    visited[0] = 1;
    int count = 1;
    while (!stack.empty()) {
        int r = stack.back(); stack.pop_back();
        for (int k : rooms[r]) {
            if (!visited[k]) { visited[k] = 1; ++count; stack.push_back(k); }
        }
    }
    return count == n;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> r{{1},{2},{3},{}}; if (canVisitAllRooms(r) != true) { std::puts("case1"); return 1; } }
    { vector<vector<int>> r{{1,3},{3,0,1},{2},{0}}; if (canVisitAllRooms(r) != false) { std::puts("case2"); return 1; } }
    { vector<vector<int>> r{{1},{}}; if (canVisitAllRooms(r) != true) { std::puts("case3"); return 1; } }
    { vector<vector<int>> r{{},{}}; if (canVisitAllRooms(r) != false) { std::puts("case4"); return 1; } }
    { vector<vector<int>> r{{1,2,3},{},{},{}}; if (canVisitAllRooms(r) != true) { std::puts("case5"); return 1; } }
    { vector<vector<int>> r{{2},{},{1},{}}; if (canVisitAllRooms(r) != false) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Treat rooms as nodes and keys as directed edges. Starting from room 0, perform a DFS/BFS, marking each newly opened room visited and pushing its keys. After the traversal, you can visit all rooms exactly when the number of distinct rooms reached equals `n`. Each room and key is processed once, giving O(n + K) time where K is the total number of keys, and O(n) space for the visited set and stack.
