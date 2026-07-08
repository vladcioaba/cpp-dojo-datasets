## challenge: Implement Queue using Stacks
tags: stack, design, queue
track: faang
difficulty: easy

Implement a first-in-first-out (FIFO) queue using only two stacks. The `MyQueue` class supports `push(x)` to add an element to the back, `pop()` to remove and return the front element, `peek()` to return the front element without removing it, and `empty()` to report whether the queue is empty. Use only the standard stack operations (push to top, pop/peek from top, size/empty).

Constraints: `1 <= x <= 9`; at most `100` calls are made across `push`, `pop`, `peek`, and `empty`; `pop` and `peek` are only called on a non-empty queue.

Example: push `1`, push `2`, `peek()` → `1`, `pop()` → `1`, `empty()` → `false`.

hint: A single stack reverses order; the front of the queue is the bottom of a push-stack.
hint: Use an input stack for pushes and an output stack that holds elements in front-to-back order.
hint: Only refill the output stack (by draining the input stack into it) when the output stack is empty — this makes each element move at most twice, so operations are amortized O(1).

```cpp
// starter
class MyQueue {
public:
    MyQueue();
    void push(int x);
    int pop();
    int peek();
    bool empty();
};
```

```cpp
class MyQueue {
    std::vector<int> in, out;
    void transfer() {
        if (out.empty())
            while (!in.empty()) { out.push_back(in.back()); in.pop_back(); }
    }
public:
    MyQueue() {}
    void push(int x) { in.push_back(x); }
    int pop() { transfer(); int v = out.back(); out.pop_back(); return v; }
    int peek() { transfer(); return out.back(); }
    bool empty() { return in.empty() && out.empty(); }
};
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { MyQueue q; q.push(1); q.push(2);
      if (q.peek() != 1)  { std::puts("case1"); return 1; }
      if (q.pop() != 1)   { std::puts("case2"); return 1; }
      if (q.empty())      { std::puts("case3"); return 1; }
      if (q.pop() != 2)   { std::puts("case4"); return 1; }
      if (!q.empty())     { std::puts("case5"); return 1; } }
    { MyQueue q; q.push(3); q.push(4); q.push(5);
      if (q.pop() != 3)   { std::puts("case6"); return 1; }
      q.push(6);
      if (q.peek() != 4)  { std::puts("case7"); return 1; }
      if (q.pop() != 4)   { std::puts("case8"); return 1; }
      if (q.pop() != 5)   { std::puts("case9"); return 1; }
      if (q.pop() != 6)   { std::puts("case10"); return 1; }
      if (!q.empty())     { std::puts("case11"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maintain an input stack for incoming elements and an output stack that stores elements in dequeue order. `push` always targets the input stack. For `pop`/`peek`, if the output stack is empty, drain the entire input stack into it — reversing the order so the queue's front sits on top. Each element is pushed and popped at most twice total, giving amortized O(1) per operation and O(n) space.
