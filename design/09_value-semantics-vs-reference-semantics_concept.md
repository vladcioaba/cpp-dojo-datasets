## fact: Value semantics vs reference semantics
tags: value-semantics, copies, aliasing
track: design

C++ lets a class behave like `int` — a **value type**: copies are independent, there is no aliasing, and reasoning is local. Java-style **reference semantics** (shared objects reached through pointers) invites aliasing bugs: a mutation through one handle is silently visible through another, and lifetime becomes a runtime puzzle.

Default to value semantics. Reach for shared/reference semantics only when the entity genuinely has one identity that many parties must observe. Regular value types compose cleanly, live in containers, and are trivially thread-confined.

```cpp
struct Point { int x, y; };
Point a{1, 2};
Point b = a;      // independent copy
b.x = 9;          // a is untouched — no aliasing

auto p = std::make_shared<Point>(1, 2);
auto q = p;       // q and p alias the SAME object
q->x = 9;         // p->x is now 9 too — spooky action at a distance
```
