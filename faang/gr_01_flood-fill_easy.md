## challenge: Flood Fill

tags: graph, dfs, matrix, flood-fill
track: faang
difficulty: easy

You are given an `m x n` integer grid `image` where `image[r][c]` is the pixel value, plus a starting pixel `(sr, sc)` and a new `color`. Perform a flood fill: starting from `(sr, sc)`, recolor that pixel and every pixel 4-directionally connected to it that shares the *original* color of the starting pixel. Return the modified image.

Constraints: `1 <= m, n <= 50`, `0 <= image[r][c], color < 2^16`, `0 <= sr < m`, `0 <= sc < n`.

Example: `image = [[1,1,1],[1,1,0],[1,0,1]], sr = 1, sc = 1, color = 2` → `[[2,2,2],[2,2,0],[2,0,1]]`. Example: if the new color equals the starting pixel's color, the image is returned unchanged.

hint: Every pixel you should recolor is reachable from the start through neighbours that all hold the same original color — that is a connected component.
hint: Record the original color before you overwrite anything, then DFS/BFS outward recoloring cells that still match it.
hint: If the new color already equals the original color, do nothing — otherwise your fill would loop forever revisiting cells.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> floodFill(std::vector<std::vector<int>>& image, int sr, int sc, int color);
```

```cpp
std::vector<std::vector<int>> floodFill(std::vector<std::vector<int>>& image, int sr, int sc, int color) {
    int m = (int)image.size(), n = (int)image[0].size();
    int start = image[sr][sc];
    if (start == color) return image;
    std::function<void(int, int)> dfs = [&](int r, int c) {
        if (r < 0 || c < 0 || r >= m || c >= n || image[r][c] != start) return;
        image[r][c] = color;
        dfs(r + 1, c); dfs(r - 1, c);
        dfs(r, c + 1); dfs(r, c - 1);
    };
    dfs(sr, sc);
    return image;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <functional>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> img{{1,1,1},{1,1,0},{1,0,1}};
      auto r = floodFill(img, 1, 1, 2);
      vector<vector<int>> exp{{2,2,2},{2,2,0},{2,0,1}};
      if (r != exp) { std::puts("case1"); return 1; } }
    { vector<vector<int>> img{{0,0,0},{0,0,0}};
      auto r = floodFill(img, 0, 0, 0);
      vector<vector<int>> exp{{0,0,0},{0,0,0}};
      if (r != exp) { std::puts("case2"); return 1; } }
    { vector<vector<int>> img{{0,0,0},{0,1,1}};
      auto r = floodFill(img, 1, 1, 2);
      vector<vector<int>> exp{{0,0,0},{0,2,2}};
      if (r != exp) { std::puts("case3"); return 1; } }
    { vector<vector<int>> img{{5}};
      auto r = floodFill(img, 0, 0, 9);
      vector<vector<int>> exp{{9}};
      if (r != exp) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Capture the source pixel's original color first; if it already equals the target color return immediately to avoid an infinite recursion. Otherwise DFS (or BFS) from the source, recoloring any 4-neighbour that still matches the original color and recursing from it. Each cell is visited a constant number of times. O(m*n) time, O(m*n) worst-case stack space.
