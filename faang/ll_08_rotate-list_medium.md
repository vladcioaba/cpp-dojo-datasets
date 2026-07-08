## challenge: Rotate List
tags: linked-list, two-pointers
track: faang
difficulty: medium

Given the `head` of a singly linked list, rotate the list to the right by `k` places and return the new head. Rotating right by one moves the last node to the front.

Constraints: the list has `0 <= n <= 500` nodes, `-100 <= Node.val <= 100`, `0 <= k <= 2 * 10^9`.

Example: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2` → `4 -> 5 -> 1 -> 2 -> 3`. Example: `0 -> 1 -> 2`, `k = 4` → `2 -> 0 -> 1`. Example: empty list, `k = 3` → empty list.

hint: Rotating by the list length returns the original list, so only `k mod n` actually matters — and `k` can far exceed `n`.
hint: First measure the length; close the list into a ring by pointing the tail at the head.
hint: The new tail is `n - (k mod n)` steps from the head; walk there, set the node after it as the new head, and break the ring by nulling the new tail's `next`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* rotateRight(ListNode* head, int k);
```

```cpp
ListNode* rotateRight(ListNode* head, int k) {
    if (!head || !head->next || k == 0) return head;
    int len = 1;
    ListNode* tail = head;
    while (tail->next) { tail = tail->next; ++len; }
    k %= len;
    if (k == 0) return head;
    tail->next = head;                 // form a ring
    int stepsToNewTail = len - k;
    ListNode* newTail = head;
    for (int i = 1; i < stepsToNewTail; ++i) newTail = newTail->next;
    ListNode* newHead = newTail->next;
    newTail->next = nullptr;
    return newHead;
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
    if (toVec(rotateRight(build({1,2,3,4,5}), 2)) != vector<int>({4,5,1,2,3})) { std::puts("case1"); return 1; }
    if (toVec(rotateRight(build({0,1,2}), 4))     != vector<int>({2,0,1}))     { std::puts("case2"); return 1; }
    if (toVec(rotateRight(build({}), 3))          != vector<int>({}))          { std::puts("case3"); return 1; }
    if (toVec(rotateRight(build({1,2}), 3))       != vector<int>({2,1}))       { std::puts("case4"); return 1; }
    if (toVec(rotateRight(build({1,2,3}), 3))     != vector<int>({1,2,3}))     { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A rotation by `k` is periodic with period `n`, so reduce `k` modulo the length (measured in one pass). Splicing the tail onto the head forms a ring; the new head sits `k` nodes from the end, equivalently the new tail is `n - k` steps from the front. Walk to that node, cut the ring there, and return its successor. O(n) time, O(1) space.
