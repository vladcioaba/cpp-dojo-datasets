## challenge: Odd Even Linked List
tags: linked-list
track: faang
difficulty: medium

Given the `head` of a singly linked list, group all nodes at odd positions together followed by all nodes at even positions, and return the reordered list. Positions are 1-indexed by their place in the original list (not by value). Preserve the relative order within each group and use O(1) extra space in O(n) time.

Constraints: the list has `0 <= n <= 10^4` nodes, `-10^6 <= Node.val <= 10^6`.

Example: `1 -> 2 -> 3 -> 4 -> 5` → `1 -> 3 -> 5 -> 2 -> 4`. Example: `2 -> 1 -> 3 -> 5 -> 6 -> 4 -> 7` → `2 -> 3 -> 6 -> 7 -> 1 -> 5 -> 4`. Example: empty list → empty list.

hint: Build two chains as you scan — one threading the odd-position nodes, one threading the even-position nodes — then join them.
hint: Keep an `odd` tail and an `even` tail; remember the head of the even chain so you can attach it after the odd chain.
hint: At each step do `odd->next = even->next` then advance odd, `even->next = odd->next` then advance even; stop when `even` or `even->next` is null, and finish with `odd->next = evenHead`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* oddEvenList(ListNode* head);
```

```cpp
ListNode* oddEvenList(ListNode* head) {
    if (!head) return head;
    ListNode* odd = head;
    ListNode* even = head->next;
    ListNode* evenHead = even;
    while (even && even->next) {
        odd->next = even->next;
        odd = odd->next;
        even->next = odd->next;
        even = even->next;
    }
    odd->next = evenHead;
    return head;
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
    if (toVec(oddEvenList(build({1,2,3,4,5})))       != vector<int>({1,3,5,2,4}))       { std::puts("case1"); return 1; }
    if (toVec(oddEvenList(build({2,1,3,5,6,4,7})))   != vector<int>({2,3,6,7,1,5,4}))   { std::puts("case2"); return 1; }
    if (toVec(oddEvenList(build({})))                != vector<int>({}))                { std::puts("case3"); return 1; }
    if (toVec(oddEvenList(build({1,2})))             != vector<int>({1,2}))             { std::puts("case4"); return 1; }
    if (toVec(oddEvenList(build({1})))               != vector<int>({1}))               { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Weave two sublists in a single pass: the odd tail repeatedly grabs `even->next` (the next odd-position node) and the even tail grabs the node after that. Saving `evenHead` lets you splice the even chain onto the odd chain's end once the loop stops. Only a handful of pointers are reused, so it is O(n) time and O(1) space.
