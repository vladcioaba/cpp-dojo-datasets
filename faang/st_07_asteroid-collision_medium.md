## challenge: Asteroid Collision
tags: stack, array, simulation
track: faang
difficulty: medium

We are given an array `asteroids` of integers representing asteroids in a row. The absolute value is an asteroid's size and the sign is its direction: positive means moving right, negative means moving left; all move at the same speed. When two asteroids meet, the smaller one explodes; if both are the same size, both explode. Two asteroids moving in the same direction, or a left-mover to the left of a right-mover, never meet. Return the state of the asteroids after all collisions.

Constraints: `2 <= asteroids.length <= 10^4`; `-1000 <= asteroids[i] <= 1000`; `asteroids[i] != 0`.

Example: `asteroids = [5,10,-5]` → `[5,10]` (the `10` and `-5` collide, `-5` explodes). Example: `asteroids = [8,-8]` → `[]`. Example: `asteroids = [10,2,-5]` → `[10]`.

hint: A collision only happens when a right-moving asteroid is immediately followed by a left-moving one.
hint: Keep surviving asteroids on a stack; a new left-mover may collide with the right-movers on top of it.
hint: Repeatedly compare the incoming left-mover to the stack top: pop smaller right-movers, stop if the top is larger, and if sizes are equal both are destroyed.

```cpp
// starter
#include <vector>
std::vector<int> asteroidCollision(std::vector<int>& asteroids);
```

```cpp
std::vector<int> asteroidCollision(std::vector<int>& asteroids) {
    std::vector<int> st;
    for (int a : asteroids) {
        bool alive = true;
        while (alive && a < 0 && !st.empty() && st.back() > 0) {
            if (st.back() < -a) { st.pop_back(); continue; } // top explodes, keep checking
            if (st.back() == -a) { st.pop_back(); }          // both explode
            alive = false;                                   // a does not survive this collision
        }
        if (alive) st.push_back(a);
    }
    return st;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{5,10,-5};    if (asteroidCollision(a) != vector<int>({5,10})) { std::puts("case1"); return 1; } }
    { vector<int> a{8,-8};       if (asteroidCollision(a) != vector<int>({}))     { std::puts("case2"); return 1; } }
    { vector<int> a{10,2,-5};    if (asteroidCollision(a) != vector<int>({10}))   { std::puts("case3"); return 1; } }
    { vector<int> a{-2,-1,1,2};  if (asteroidCollision(a) != vector<int>({-2,-1,1,2})) { std::puts("case4"); return 1; } }
    { vector<int> a{-2,-2,1,-2};  if (asteroidCollision(a) != vector<int>({-2,-2,-2})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Simulate with a stack of surviving asteroids. Each asteroid is pushed unless it is a left-mover facing right-movers on top of the stack, in which case collisions resolve on the top: a smaller right-mover is popped and the left-mover continues; an equal right-mover cancels the left-mover and is popped; a larger right-mover destroys the left-mover. Only when it survives every collision is the left-mover pushed. Each asteroid is pushed and popped at most once, so the total work is O(n) time and O(n) space.
