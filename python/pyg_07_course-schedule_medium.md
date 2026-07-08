## challenge: Course Schedule
tags: graph, topological-sort, dfs, bfs
track: python
lang: python
difficulty: medium

There are `num_courses` courses labelled `0` to `num_courses - 1`. Each pair `[a, b]` in `prerequisites` means you must take course `b` before course `a`. Return `True` if you can finish every course, `False` otherwise.

Constraints: `1 <= num_courses <= 2000`, `0 <= len(prerequisites) <= 5000`, all prerequisite pairs are distinct.

Example: `num_courses = 2, prerequisites = [[1, 0]]` → `True`. With `[[1, 0], [0, 1]]` → `False` (the two courses depend on each other).

hint: You can finish all courses exactly when the prerequisite graph has no cycle.
hint: Kahn's algorithm: repeatedly remove a course whose remaining in-degree is 0.
hint: If you manage to remove all `num_courses` nodes this way, there was no cycle.

```python
# starter
def can_finish(num_courses, prerequisites):
    ...
```

```python
def can_finish(num_courses, prerequisites):
    from collections import deque
    graph = [[] for _ in range(num_courses)]
    indeg = [0] * num_courses
    for a, b in prerequisites:
        graph[b].append(a)
        indeg[a] += 1
    q = deque(i for i in range(num_courses) if indeg[i] == 0)
    removed = 0
    while q:
        node = q.popleft()
        removed += 1
        for nxt in graph[node]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                q.append(nxt)
    return removed == num_courses
```

```python
# harness
#__USER__
def _check():
    assert can_finish(2, [[1, 0]]) is True
    assert can_finish(2, [[1, 0], [0, 1]]) is False
    assert can_finish(1, []) is True
    assert can_finish(4, [[1, 0], [2, 1], [3, 2]]) is True
    assert can_finish(3, [[0, 1], [1, 2], [2, 0]]) is False
    assert can_finish(5, [[1, 0], [2, 0], [3, 1], [3, 2], [4, 3]]) is True
    print("PASS")

_check()
```

**Editorial:** Model courses as a directed graph where an edge `b → a` means "b unlocks a". A valid schedule exists iff the graph is a DAG. Kahn's topological sort peels off zero-in-degree nodes; if every node is peeled, there is no cycle. O(V + E) time and space.
