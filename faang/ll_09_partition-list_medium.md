## challenge: Partition List
tags: linked-list, two-pointers
track: faang
difficulty: medium

Given the `head` of a singly linked list and a value `x`, partition it so that all nodes with value strictly less than `x` come before all nodes with value greater than or equal to `x`. You must preserve the original relative order of the nodes within each of the two partitions. Return the head of the reordered list.

Constraints: the list has `0 <= n <= 200` nodes, `-100 <= Node.val <= 100`, `-200 <= x <= 200`.

Example: `1 -> 4 -> 3 -> 2 -> 5 -> 2`, `x = 3` → `1 -> 2 -> 2 -> 4 -> 3 -> 5`. Example: `2 -> 1`, `x = 2` → `1 -> 2`. Example: empty list, `x = 0` → empty list.

hint: Think of sorting the nodes into two buckets while walking the list once, never reordering within a bucket.
hint: Maintain two separate chains with their own dummy heads — a "less than x" chain and a "greater-or-equal" chain — appending each node to the right one.
hint: After the scan, terminate the greater-or-equal chain with a null `next` (crucial to avoid a cycle) and link the tail of the less chain to the head of the greater-or-equal chain.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* partition(ListNode* head, int x);
```

```cpp
ListNode* partition(ListNode* head, int x) {
    ListNode lessDummy(0), geDummy(0);
    ListNode* less = &lessDummy;
    ListNode* ge = &geDummy;
    while (head) {
        if (head->val < x) { less->next = head; less = less->next; }
        else               { ge->next = head;   ge = ge->next; }
        head = head->next;
    }
    ge->next = nullptr;
    less->next = geDummy.next;
    return lessDummy.next;
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
    if (toVec(partition(build({1,4,3,2,5,2}), 3)) != vector<int>({1,2,2,4,3,5})) { std::puts("case1"); return 1; }
    if (toVec(partition(build({2,1}), 2))         != vector<int>({1,2}))         { std::puts("case2"); return 1; }
    if (toVec(partition(build({}), 0))            != vector<int>({}))            { std::puts("case3"); return 1; }
    if (toVec(partition(build({1,1}), 0))         != vector<int>({1,1}))         { std::puts("case4"); return 1; }
    if (toVec(partition(build({3,1,2}), 3))       != vector<int>({1,2,3}))       { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Thread two independent lists in a single pass, each anchored by a dummy head: nodes below `x` join the "less" chain, the rest join the "greater-or-equal" chain, and appending to a tail preserves original order. Null-terminate the second chain (otherwise the last original node could form a cycle), then join the less-tail to the ge-head. O(n) time, O(1) extra space.
