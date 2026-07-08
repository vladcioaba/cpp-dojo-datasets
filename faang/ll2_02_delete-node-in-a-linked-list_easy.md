## challenge: Delete Node in a Linked List
tags: linked-list
track: faang
difficulty: easy

You are given direct access to a single `node` in a singly linked list, and this `node` is guaranteed not to be the last node. You are **not** given the head. Delete that node from the list — after your call the list must look as if the node had never existed, with every other node keeping its value and relative order. All values in the list are unique.

Constraints: the list has `2 <= n <= 1000` nodes, `-1000 <= Node.val <= 1000`, values are unique, and the target node is not the tail.

Example: `4 -> 5 -> 1 -> 9`, delete the node holding `5` → `4 -> 1 -> 9`. Example: `4 -> 5 -> 1 -> 9`, delete the node holding `1` → `4 -> 5 -> 9`.

hint: You can't reach the previous node to unlink this one — but you can make this node *impersonate* the one after it.
hint: Copy the next node's value into the current node, then splice out the next node with `node->next = node->next->next`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
void deleteNode(ListNode* node);
```

```cpp
void deleteNode(ListNode* node) {
    node->val = node->next->val;
    node->next = node->next->next;
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
static vector<int> toVec(ListNode* h) {
    vector<int> out; while (h) { out.push_back(h->val); h = h->next; } return out;
}
static ListNode* find(ListNode* h, int val) {
    while (h && h->val != val) h = h->next; return h;
}
//__USER__
int main() {
    { ListNode* h = build({4,5,1,9}); deleteNode(find(h,5)); if (toVec(h) != vector<int>({4,1,9})) { std::puts("case1"); return 1; } }
    { ListNode* h = build({4,5,1,9}); deleteNode(find(h,1)); if (toVec(h) != vector<int>({4,5,9})) { std::puts("case2"); return 1; } }
    { ListNode* h = build({1,2});     deleteNode(find(h,1)); if (toVec(h) != vector<int>({2}))     { std::puts("case3"); return 1; } }
    { ListNode* h = build({0,-1,3,7}); deleteNode(find(h,3)); if (toVec(h) != vector<int>({0,-1,7})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Without the predecessor you cannot physically remove this node's cell, but deletion only needs to change what the list *looks* like. Overwrite the node's value with its successor's, then bypass the successor by pointing `next` past it. The successor is effectively removed and this node now carries its data, which is why the guarantee that it is not the tail matters. O(1) time, O(1) space.
