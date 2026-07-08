## challenge: Reorder List
tags: linked-list, two-pointers, stack
track: faang
difficulty: hard

Given the `head` of a singly linked list `L0 -> L1 -> ... -> Ln-1 -> Ln`, reorder it in place to `L0 -> Ln -> L1 -> Ln-1 -> L2 -> Ln-2 -> ...`. You may not modify the node values — only rewire pointers — and it must run in O(n) time with O(1) extra space. The function reorders the list rooted at `head` (which stays the first node).

Constraints: the list has `1 <= n <= 5 * 10^4` nodes, `1 <= Node.val <= 1000`.

Example: `1 -> 2 -> 3 -> 4` → `1 -> 4 -> 2 -> 3`. Example: `1 -> 2 -> 3 -> 4 -> 5` → `1 -> 5 -> 2 -> 4 -> 3`. Example: `1` → `1`.

hint: The result interleaves the front half with the back half read backwards — think three sub-problems, not one.
hint: Find the middle with slow/fast pointers, then reverse the second half in place so you can read it front to back.
hint: Merge the two halves by alternately taking one node from each, carefully saving each `next` before you overwrite it so you never lose the remainder of either half.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
void reorderList(ListNode* head);
```

```cpp
void reorderList(ListNode* head) {
    if (!head || !head->next) return;
    // 1. find the middle (slow ends on the first-half tail)
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    // 2. reverse the second half
    ListNode* second = slow->next;
    slow->next = nullptr;
    ListNode* prev = nullptr;
    while (second) {
        ListNode* nxt = second->next;
        second->next = prev;
        prev = second;
        second = nxt;
    }
    // 3. merge the two halves alternately
    ListNode* first = head;
    second = prev;
    while (second) {
        ListNode* fnext = first->next;
        ListNode* snext = second->next;
        first->next = second;
        second->next = fnext;
        first = fnext;
        second = snext;
    }
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
    { ListNode* h = build({1,2,3,4});   reorderList(h); if (toVec(h) != vector<int>({1,4,2,3}))   { std::puts("case1"); return 1; } }
    { ListNode* h = build({1,2,3,4,5}); reorderList(h); if (toVec(h) != vector<int>({1,5,2,4,3})) { std::puts("case2"); return 1; } }
    { ListNode* h = build({1});         reorderList(h); if (toVec(h) != vector<int>({1}))         { std::puts("case3"); return 1; } }
    { ListNode* h = build({1,2});       reorderList(h); if (toVec(h) != vector<int>({1,2}))       { std::puts("case4"); return 1; } }
    { ListNode* h = build({1,2,3});     reorderList(h); if (toVec(h) != vector<int>({1,3,2}))     { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Decompose into three linear passes. Slow/fast pointers locate the midpoint and split the list; reversing the second half lets it be consumed from its original tail forward. A final alternating merge weaves one node from each half, always caching the successor pointers before rewiring so neither remainder is lost. Each phase is O(n) and uses only a constant number of pointers, giving O(n) time and O(1) space.
