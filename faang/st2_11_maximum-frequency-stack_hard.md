## challenge: Maximum Frequency Stack
tags: stack, design, hash-table
track: faang
difficulty: hard

Design a stack-like data structure `FreqStack` that pushes and pops elements by frequency. Implement `push(val)`, which adds an integer, and `pop()`, which removes and returns the most frequent element. If several elements share the highest frequency, `pop` returns the one that was pushed most recently among them.

Constraints: `0 <= val <= 10^9`; at most `2 * 10^4` calls are made to `push` and `pop` combined; every call to `pop` happens on a non-empty stack.

Example: push `5, 7, 5, 7, 4, 5`, then four `pop()` calls return `5, 7, 5, 4`. (`5` has frequency 3; then `5` and `7` are tied at 2 so the more recent `7` wins; then `5`; then `4`.)

hint: The element to pop depends on its current frequency and, among ties, recency — so group elements by how many times they have been pushed.
hint: Keep a per-value frequency count and, for each frequency level, a stack of the values that have reached that level.
hint: `pop` takes from the stack at the current maximum frequency; when that stack empties, the maximum frequency drops by one.

```cpp
// starter
class FreqStack {
public:
    FreqStack();
    void push(int val);
    int pop();
};
```

```cpp
class FreqStack {
    std::unordered_map<int, int> freq;                 // value -> current count
    std::unordered_map<int, std::vector<int>> group;   // frequency -> stack of values
    int maxFreq = 0;
public:
    FreqStack() {}
    void push(int val) {
        int f = ++freq[val];
        if (f > maxFreq) maxFreq = f;
        group[f].push_back(val);
    }
    int pop() {
        int val = group[maxFreq].back();
        group[maxFreq].pop_back();
        --freq[val];
        if (group[maxFreq].empty()) --maxFreq;
        return val;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <unordered_map>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { FreqStack fs;
      fs.push(5); fs.push(7); fs.push(5); fs.push(7); fs.push(4); fs.push(5);
      if (fs.pop() != 5) { std::puts("case1"); return 1; }
      if (fs.pop() != 7) { std::puts("case2"); return 1; }
      if (fs.pop() != 5) { std::puts("case3"); return 1; }
      if (fs.pop() != 4) { std::puts("case4"); return 1; } }
    { FreqStack fs;
      fs.push(1); fs.push(1); fs.push(2);
      if (fs.pop() != 1) { std::puts("case5"); return 1; }
      fs.push(2); fs.push(2);
      if (fs.pop() != 2) { std::puts("case6"); return 1; }
      if (fs.pop() != 2) { std::puts("case7"); return 1; }
      if (fs.pop() != 2) { std::puts("case8"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Bucket values by their frequency. A hash map `freq` records how many times each value is currently present, and a second map `group` maps each frequency level to a stack of the values that have reached it (in push order). On `push(val)`, increment its frequency to `f`, update `maxFreq`, and append `val` to `group[f]`. On `pop`, take the top of `group[maxFreq]` — this is automatically the most-frequent, most-recently-pushed element — decrement its frequency, and drop `maxFreq` when that bucket empties. Both operations are O(1) amortized, with O(n) total space.
