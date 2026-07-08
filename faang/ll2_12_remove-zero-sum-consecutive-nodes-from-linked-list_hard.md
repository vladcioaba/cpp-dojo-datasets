## challenge: Remove Zero Sum Consecutive Nodes from Linked List
tags: linked-list, hash-table, prefix-sum
track: faang
difficulty: hard

Given the `head` of a singly linked list, repeatedly delete consecutive sequences of nodes that sum to `0` until no such sequence remains, then return the head of the final list. The answer is unique for the given inputs (the order of deletions does not affect it).

Constraints: the list has `1 <= n <= 1000` nodes, `-1000 <= Node.val <= 1000`.

Example: `1 -> 2 -> -3 -> 3 -> 1` → `3 -> 1` (the prefix `1, 2, -3` sums to 0). Example: `1 -> 2 -> 3 -> -3 -> 4` → `1 -> 2 -> 4`. Example: `1 -> 2 -> 3 -> -3 -> -2` → `1`.

hint: A consecutive run `nodes[i..j]` sums to zero exactly when the running prefix sum before `i` equals the running prefix sum at `j` — so equal prefix sums bracket a removable block.
hint: Use a dummy head and a map from prefix sum to the *last* node achieving it; on a second pass, whenever a prefix sum repeats, splice out everything between the two nodes by pointing the earlier node's `next` at the later node's `next`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* removeZeroSumSublists(ListNode* head);
```

```cpp
ListNode* removeZeroSumSublists(ListNode* head) {
    ListNode dummy(0);
    dummy.next = head;
    std::unordered_map<int, ListNode*> lastAt;
    int prefix = 0;
    // first pass: remember the last node reaching each prefix sum
    for (ListNode* p = &dummy; p; p = p->next) {
        prefix += p->val;
        lastAt[prefix] = p;
    }
    // second pass: jump over every zero-sum block
    prefix = 0;
    for (ListNode* p = &dummy; p; p = p->next) {
        prefix += p->val;
        p->next = lastAt[prefix]->next;
    }
    return dummy.next;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
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
    if (toVec(removeZeroSumSublists(build({1,2,-3,3,1})))  != vector<int>({3,1}))   { std::puts("case1"); return 1; }
    if (toVec(removeZeroSumSublists(build({1,2,3,-3,4})))  != vector<int>({1,2,4})) { std::puts("case2"); return 1; }
    if (toVec(removeZeroSumSublists(build({1,2,3,-3,-2}))) != vector<int>({1}))     { std::puts("case3"); return 1; }
    if (toVec(removeZeroSumSublists(build({1,-1})))        != vector<int>({}))      { std::puts("case4"); return 1; }
    if (toVec(removeZeroSumSublists(build({5,-3,-2,1,2,-1}))) != vector<int>({1,2,-1})) { std::puts("case5"); return 1; }
    if (toVec(removeZeroSumSublists(build({2,2,-2,-2,3}))) != vector<int>({3}))     { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Think in prefix sums over a dummy-prefixed list. If the running sum has the same value at two different nodes, the segment strictly between them totals zero and can be removed. In one pass, record for each prefix-sum value the last node that reaches it. In a second pass, recompute the prefix sum and, at each node, jump directly to `lastAt[prefix]->next` — this hops over the longest zero-sum block ending at that value and, because we stored the last occurrence, correctly resolves nested and overlapping cancellations in a single sweep. Two O(n) passes with an O(n) hash map, so O(n) time and space.
