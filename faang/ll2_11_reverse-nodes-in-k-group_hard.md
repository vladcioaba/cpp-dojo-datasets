## challenge: Reverse Nodes in k-Group
tags: linked-list, recursion
track: faang
difficulty: hard

Given the `head` of a singly linked list, reverse the nodes of the list `k` at a time and return the modified list. `k` is a positive integer no larger than the list length. If the number of nodes is not a multiple of `k`, the final leftover nodes stay in their original order. You may not alter the node values — only rewire pointers.

Constraints: the list has `1 <= n <= 5000` nodes, `0 <= Node.val <= 1000`, `1 <= k <= n`.

Example: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2` → `2 -> 1 -> 4 -> 3 -> 5`. Example: `1 -> 2 -> 3 -> 4 -> 5`, `k = 3` → `3 -> 2 -> 1 -> 4 -> 5`. Example: `1 -> 2 -> 3 -> 4 -> 5`, `k = 1` → `1 -> 2 -> 3 -> 4 -> 5`.

hint: Before reversing a block, confirm there are actually `k` nodes ahead — if not, leave that tail untouched.
hint: Reverse the first `k` nodes, but connect their (originally) first node to the already-reversed result of the *rest* of the list; recursion makes this connection clean.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* reverseKGroup(ListNode* head, int k);
```

```cpp
ListNode* reverseKGroup(ListNode* head, int k) {
    // check there are at least k nodes
    ListNode* node = head;
    for (int i = 0; i < k; ++i) {
        if (!node) return head;
        node = node->next;
    }
    // node is now the head of the remainder; reverse it first
    ListNode* prev = reverseKGroup(node, k);
    // reverse this group of k, appending onto the reversed remainder
    ListNode* cur = head;
    for (int i = 0; i < k; ++i) {
        ListNode* nxt = cur->next;
        cur->next = prev;
        prev = cur;
        cur = nxt;
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
    if (toVec(reverseKGroup(build({1,2,3,4,5}), 2)) != vector<int>({2,1,4,3,5})) { std::puts("case1"); return 1; }
    if (toVec(reverseKGroup(build({1,2,3,4,5}), 3)) != vector<int>({3,2,1,4,5})) { std::puts("case2"); return 1; }
    if (toVec(reverseKGroup(build({1,2,3,4,5}), 1)) != vector<int>({1,2,3,4,5})) { std::puts("case3"); return 1; }
    if (toVec(reverseKGroup(build({1,2,3,4,5,6}), 3)) != vector<int>({3,2,1,6,5,4})) { std::puts("case4"); return 1; }
    if (toVec(reverseKGroup(build({1,2,3,4,5}), 5)) != vector<int>({5,4,3,2,1})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Process one group at a time. First scan `k` nodes ahead: if the list runs out before reaching `k`, this trailing block is shorter than `k`, so return it unchanged. Otherwise recurse on the node just past the group to obtain the fully reversed remainder, then reverse this group's `k` nodes with the standard prev/cur pointer flip, seeding `prev` with that reversed remainder so the group's original first node links straight into it. The new group head is returned upward. Each node is visited a constant number of times across the counting and reversing passes, giving O(n) time and O(n / k) recursion depth.
