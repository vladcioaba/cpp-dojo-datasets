## challenge: Palindrome Linked List
tags: linked-list, two-pointers, stack
track: faang
difficulty: hard

Given the `head` of a singly linked list, return `true` if the sequence of node values reads the same forwards and backwards, and `false` otherwise. Solve it in O(n) time using O(1) extra space (no copying the values into an array).

Constraints: the list has `0 <= n <= 10^5` nodes, `0 <= Node.val <= 9`.

Example: `1 -> 2 -> 2 -> 1` → `true`. Example: `1 -> 2 -> 3 -> 2 -> 1` → `true`. Example: `1 -> 2` → `false`. Example: empty list → `true`.

hint: Copying to a vector and checking symmetry is O(n) space; the O(1) trick reuses the list's own pointers.
hint: Find the middle with slow/fast pointers, then reverse the second half in place.
hint: Walk the first half and the reversed second half in tandem comparing values; they match iff the list is a palindrome (the odd middle element never needs comparing).

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
bool isPalindrome(ListNode* head);
```

```cpp
bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;
    // find the middle
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    // reverse the second half
    ListNode* second = slow->next;
    ListNode* prev = nullptr;
    while (second) {
        ListNode* nxt = second->next;
        second->next = prev;
        prev = second;
        second = nxt;
    }
    // compare halves
    ListNode* p1 = head;
    ListNode* p2 = prev;
    bool ok = true;
    while (p2) {
        if (p1->val != p2->val) { ok = false; break; }
        p1 = p1->next;
        p2 = p2->next;
    }
    return ok;
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
    if (isPalindrome(build({1,2,2,1}))   != true)  { std::puts("case1"); return 1; }
    if (isPalindrome(build({1,2,3,2,1})) != true)  { std::puts("case2"); return 1; }
    if (isPalindrome(build({1,2}))       != false) { std::puts("case3"); return 1; }
    if (isPalindrome(build({}))          != true)  { std::puts("case4"); return 1; }
    if (isPalindrome(build({1,2,3}))     != false) { std::puts("case5"); return 1; }
    if (isPalindrome(build({7}))         != true)  { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Locate the midpoint with slow/fast pointers, reverse the trailing half in place, then compare it node-by-node against the leading half. A palindrome matches on every step; for odd lengths the lone middle element is naturally skipped because the reversed half is shorter. Two linear passes with a constant number of pointers give O(n) time and O(1) space, unlike the array-copy approach.
