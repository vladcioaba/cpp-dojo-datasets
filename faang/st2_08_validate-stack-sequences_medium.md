## challenge: Validate Stack Sequences
tags: stack, array, simulation
track: faang
difficulty: medium

Given two integer arrays `pushed` and `popped`, each a permutation of the same distinct values, return `true` if and only if this could be the result of a sequence of `push` and `pop` operations on an initially empty stack — that is, elements are pushed in the order given by `pushed` and could be popped in the order given by `popped`.

Constraints: `1 <= pushed.length <= 1000`; `popped.length == pushed.length`; `0 <= pushed[i], popped[i] < 1000`; `pushed` is a permutation of `popped`; all elements are distinct.

Example: `pushed = [1,2,3,4,5], popped = [4,5,3,2,1]` → `true` (push 1,2,3,4, pop 4, push 5, pop 5,3,2,1). Example: `pushed = [1,2,3,4,5], popped = [4,3,5,1,2]` → `false` (`1` cannot be popped before `2`).

hint: Just simulate — actually run the pushes and greedily pop whenever the top matches the next value to be popped.
hint: Push elements one by one; after each push, pop as long as the stack top equals `popped[j]`, advancing `j`.
hint: The sequence is valid exactly when the stack is empty once every element has been pushed.

```cpp
// starter
#include <vector>
bool validateStackSequences(std::vector<int>& pushed, std::vector<int>& popped);
```

```cpp
bool validateStackSequences(std::vector<int>& pushed, std::vector<int>& popped) {
    std::vector<int> st;
    int j = 0;
    for (int x : pushed) {
        st.push_back(x);
        while (!st.empty() && j < (int)popped.size() && st.back() == popped[j]) {
            st.pop_back();
            ++j;
        }
    }
    return st.empty();
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{1,2,3,4,5}, b{4,5,3,2,1}; if (!validateStackSequences(a,b)) { std::puts("case1"); return 1; } }
    { vector<int> a{1,2,3,4,5}, b{4,3,5,1,2}; if ( validateStackSequences(a,b)) { std::puts("case2"); return 1; } }
    { vector<int> a{1}, b{1};                 if (!validateStackSequences(a,b)) { std::puts("case3"); return 1; } }
    { vector<int> a{1,2,3}, b{3,2,1};         if (!validateStackSequences(a,b)) { std::puts("case4"); return 1; } }
    { vector<int> a{2,1,0}, b{1,2,0};         if (!validateStackSequences(a,b)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Directly simulate the machine. Push values from `pushed` one at a time; after each push, greedily pop while the stack top equals the next expected value `popped[j]`, advancing `j`. This greedy popping is forced: since values are distinct, whenever the top matches the required pop it must be removed now, because it can never be popped later once buried. If, after pushing everything, the stack is empty, all pops were satisfiable and the sequence is valid. The simulation is O(n) time and O(n) space.
