## challenge: Min Stack
tags: stack, design
track: faang
difficulty: medium

Design a stack that supports `push`, `pop`, `top`, and retrieving the minimum element, each in O(1) time. Implement the `MinStack` class: `push(val)` pushes `val` onto the stack, `pop()` removes the top element, `top()` returns the top element, and `getMin()` returns the smallest element currently in the stack. All operations are called on a non-empty stack when they read a value.

Constraints: `-2^31 <= val <= 2^31 - 1`; `pop`, `top`, and `getMin` are only called on a non-empty stack; at most `3 * 10^4` calls are made in total.

Example: push `-2`, push `0`, push `-3`, `getMin()` → `-3`, `pop()`, `top()` → `0`, `getMin()` → `-2`.

hint: The tricky operation is `getMin` in O(1); scanning the stack each time would be O(n).
hint: Store, alongside each value, the minimum of the stack at the moment it was pushed.
hint: A parallel "min stack" whose top always mirrors the current minimum makes `push`/`pop`/`getMin` all O(1).

```cpp
// starter
class MinStack {
public:
    MinStack();
    void push(int val);
    void pop();
    int top();
    int getMin();
};
```

```cpp
class MinStack {
    std::vector<int> data;
    std::vector<int> mins;   // mins.back() == min of current stack
public:
    MinStack() {}
    void push(int val) {
        data.push_back(val);
        if (mins.empty() || val <= mins.back()) mins.push_back(val);
        else mins.push_back(mins.back());
    }
    void pop() {
        data.pop_back();
        mins.pop_back();
    }
    int top() { return data.back(); }
    int getMin() { return mins.back(); }
};
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { MinStack s; s.push(-2); s.push(0); s.push(-3);
      if (s.getMin() != -3) { std::puts("case1"); return 1; }
      s.pop();
      if (s.top() != 0)   { std::puts("case2"); return 1; }
      if (s.getMin() != -2) { std::puts("case3"); return 1; } }
    { MinStack s; s.push(5);
      if (s.top() != 5 || s.getMin() != 5) { std::puts("case4"); return 1; }
      s.push(3); s.push(3);
      if (s.getMin() != 3) { std::puts("case5"); return 1; }
      s.pop();
      if (s.getMin() != 3) { std::puts("case6"); return 1; }
      s.pop();
      if (s.getMin() != 5) { std::puts("case7"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Keep two parallel stacks: the values and, for each position, the running minimum at the time of that push. On `push`, append `min(val, currentMin)` to the min stack; on `pop`, discard the top of both. `top` reads the value stack and `getMin` reads the min stack, so every operation is O(1) time and the structure uses O(n) space.
