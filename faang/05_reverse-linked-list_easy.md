## challenge: Reverse Linked List
tags: linked-list, recursion
track: faang
difficulty: easy

Reverse a singly linked list and return the new head. Do it iteratively in O(n) time and O(1) extra space.

Constraints: the list has `0 <= n <= 5000` nodes, `-5000 <= Node.val <= 5000`.

Example: `1 -> 2 -> 3 -> 4 -> 5` → `5 -> 4 -> 3 -> 2 -> 1`. Example: empty list → empty list.

hint: Walk the list once, flipping each node's `next` pointer to point back at the node before it.
hint: Keep three pointers — previous, current, and a saved next — so you never lose the tail of the list.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* reverseList(ListNode* head);
```

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    while (head) {
        ListNode* nxt = head->next;
        head->next = prev;
        prev = head;
        head = nxt;
    }
    return prev;
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
//__USER__
int main() {
    if (toVec(reverseList(build({1,2,3,4,5}))) != vector<int>({5,4,3,2,1})) { std::puts("case1"); return 1; }
    if (toVec(reverseList(build({1,2})))       != vector<int>({2,1}))       { std::puts("case2"); return 1; }
    if (toVec(reverseList(build({})))          != vector<int>({}))          { std::puts("case3"); return 1; }
    if (toVec(reverseList(build({42})))        != vector<int>({42}))        { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Iterate through the list reversing each `next` pointer, saving the following node before you overwrite it. When you fall off the end, `prev` is the new head. O(n) time, O(1) space.
