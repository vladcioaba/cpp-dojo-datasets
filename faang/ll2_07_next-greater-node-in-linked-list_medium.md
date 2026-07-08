## challenge: Next Greater Node In Linked List
tags: linked-list, stack, monotonic-stack
track: faang
difficulty: medium

Given the `head` of a singly linked list, return an integer array `answer` where `answer[i]` is the value of the **first** node strictly greater than the `i`-th node when scanning to its right. If no such node exists, `answer[i]` is `0`. Nodes are numbered from `0` starting at the head.

Constraints: the list has `1 <= n <= 10^4` nodes, `1 <= Node.val <= 10^9`.

Example: `2 -> 1 -> 5` → `[5, 5, 0]`. Example: `2 -> 7 -> 4 -> 3 -> 5` → `[7, 0, 5, 5, 0]`. Example: `1 -> 7 -> 5 -> 1 -> 9 -> 2 -> 5 -> 1` → `[7, 9, 9, 9, 0, 5, 0, 0]`.

hint: This is "next greater element" over the sequence of values — a monotonic stack solves it in one pass.
hint: Copy the values into an array, then scan left to right keeping a stack of indices whose answer is still unknown; when the current value exceeds the value at the stack top, you've found that index's answer, so resolve and pop it.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
std::vector<int> nextLargerNodes(ListNode* head);
```

```cpp
std::vector<int> nextLargerNodes(ListNode* head) {
    std::vector<int> vals;
    for (ListNode* p = head; p; p = p->next) vals.push_back(p->val);
    std::vector<int> res(vals.size(), 0);
    std::vector<int> st;  // indices with no answer yet
    for (int i = 0; i < (int)vals.size(); ++i) {
        while (!st.empty() && vals[st.back()] < vals[i]) {
            res[st.back()] = vals[i];
            st.pop_back();
        }
        st.push_back(i);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
static ListNode* build(const vector<int>& v) {
    ListNode dummy(0); ListNode* t = &dummy;
    for (int x : v) { t->next = new ListNode(x); t = t->next; }
    return dummy.next;
}
//__USER__
int main() {
    if (nextLargerNodes(build({2,1,5}))         != vector<int>({5,5,0}))         { std::puts("case1"); return 1; }
    if (nextLargerNodes(build({2,7,4,3,5}))     != vector<int>({7,0,5,5,0}))     { std::puts("case2"); return 1; }
    if (nextLargerNodes(build({1,7,5,1,9,2,5,1})) != vector<int>({7,9,9,9,0,5,0,0})) { std::puts("case3"); return 1; }
    if (nextLargerNodes(build({1}))             != vector<int>({0}))             { std::puts("case4"); return 1; }
    if (nextLargerNodes(build({5,4,3,2,1}))     != vector<int>({0,0,0,0,0}))     { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Materialize the node values into an array so positions are random-access. Sweep left to right maintaining a stack of indices that are still waiting for a greater value; the values at those indices are non-increasing from bottom to top. When the current value beats the value at the top, that top index has found its next greater node, so record it and pop, repeating until the stack top is at least the current value. Push the current index and continue. Every index is pushed and popped once, giving O(n) time and O(n) space.
