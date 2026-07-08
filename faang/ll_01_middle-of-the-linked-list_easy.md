## challenge: Middle of the Linked List
tags: linked-list, two-pointers
track: faang
difficulty: easy

Given the `head` of a singly linked list, return the middle node. If there are two middle nodes (the list has an even number of nodes), return the second of them. The returned node's value and everything after it define the answer.

Constraints: the list has `1 <= n <= 100` nodes, `1 <= Node.val <= 100`.

Example: `1 -> 2 -> 3 -> 4 -> 5` → node `3` (the sublist `3 -> 4 -> 5`). Example: `1 -> 2 -> 3 -> 4 -> 5 -> 6` → node `4` (the second middle, sublist `4 -> 5 -> 6`).

hint: You could count the nodes first and then walk half of them, but a single pass is possible.
hint: Send two pointers from the head — one moving one node at a time, one moving two nodes at a time.
hint: When the fast pointer runs off the end, the slow pointer sits exactly on the middle; the even case falls out naturally when you stop on `fast && fast->next`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* middleNode(ListNode* head);
```

```cpp
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
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
    if (toVec(middleNode(build({1,2,3,4,5})))   != vector<int>({3,4,5}))   { std::puts("case1"); return 1; }
    if (toVec(middleNode(build({1,2,3,4,5,6}))) != vector<int>({4,5,6}))   { std::puts("case2"); return 1; }
    if (toVec(middleNode(build({1})))           != vector<int>({1}))       { std::puts("case3"); return 1; }
    if (toVec(middleNode(build({1,2})))         != vector<int>({2}))       { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Advance a slow pointer one node and a fast pointer two nodes each step. The fast pointer covers twice the distance, so when it reaches the end the slow pointer is halfway. Stopping while `fast && fast->next` lands slow on the second middle for even lengths. O(n) time, O(1) space, one pass, no length precomputation.
