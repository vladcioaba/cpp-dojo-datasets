## challenge: Copy List with Random Pointer
tags: linked-list, hash-table
track: faang
difficulty: hard

A linked list of length `n` is given where each node holds a `val`, a `next` pointer, and a `random` pointer that points to any node in the list or to null. Construct a deep copy: exactly `n` brand-new nodes whose `val`, `next`, and `random` mirror the original structure, where every pointer in the copy points to a node in the copy (never into the original). Return the head of the copied list.

Constraints: the list has `0 <= n <= 1000` nodes, `-10^4 <= Node.val <= 10^4`; each `random` is null or points to a node in the list.

Example: original `[[7,null],[13,0],[11,4],[10,2],[1,0]]` (each pair is `[val, index-of-random-or-null]`) → a structurally identical list of fresh nodes. Example: `[[1,1],[2,1]]` → both randoms point to the second copied node. Example: empty list → empty list.

hint: The hard part is the `random` pointer — when you copy a node you may not yet have created the node its `random` refers to.
hint: One approach maps each original node to its copy in a hash table, then a second pass wires up `next` and `random` via the map. Can you avoid the extra map?
hint: The O(1)-space trick interleaves each copy right after its original (`A -> A' -> B -> B' -> ...`); then `copy->random = original->random->next`, and finally unweave the two lists to restore the original and extract the copy.

```cpp
// starter
struct Node {
    int val;
    Node* next;
    Node* random;
    Node(int x) : val(x), next(nullptr), random(nullptr) {}
};
Node* copyRandomList(Node* head);
```

```cpp
Node* copyRandomList(Node* head) {
    if (!head) return nullptr;
    // 1. interleave a copy after each original node
    for (Node* cur = head; cur; ) {
        Node* copy = new Node(cur->val);
        copy->next = cur->next;
        cur->next = copy;
        cur = copy->next;
    }
    // 2. wire up the random pointers of the copies
    for (Node* cur = head; cur; cur = cur->next->next) {
        if (cur->random) cur->next->random = cur->random->next;
    }
    // 3. unweave to separate original and copy
    Node* newHead = head->next;
    for (Node* cur = head; cur; ) {
        Node* copy = cur->next;
        cur->next = copy->next;
        if (copy->next) copy->next = copy->next->next;
        cur = cur->next;
    }
    return newHead;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
using std::vector;
struct Node {
    int val;
    Node* next;
    Node* random;
    Node(int x) : val(x), next(nullptr), random(nullptr) {}
};
// rnd[i] = index the i-th node's random points to, or -1 for null
static Node* buildRand(const vector<int>& vals, const vector<int>& rnd) {
    if (vals.empty()) return nullptr;
    vector<Node*> nodes;
    for (int v : vals) nodes.push_back(new Node(v));
    for (size_t i = 0; i + 1 < nodes.size(); ++i) nodes[i]->next = nodes[i + 1];
    for (size_t i = 0; i < nodes.size(); ++i)
        if (rnd[i] >= 0) nodes[i]->random = nodes[rnd[i]];
    return nodes[0];
}
static bool checkCopy(Node* orig, Node* copy, const vector<int>& vals, const vector<int>& rnd) {
    vector<Node*> os, cs;
    for (Node* p = orig; p; p = p->next) os.push_back(p);
    for (Node* p = copy; p; p = p->next) cs.push_back(p);
    if (cs.size() != vals.size()) return false;
    if (os.size() != vals.size()) return false;   // original must be intact
    std::unordered_map<Node*, int> idx;
    for (int i = 0; i < (int)cs.size(); ++i) idx[cs[i]] = i;
    for (int i = 0; i < (int)cs.size(); ++i) {
        if (cs[i]->val != vals[i]) return false;
        for (Node* o : os) if (cs[i] == o) return false;  // must be a fresh node
        if (rnd[i] < 0) {
            if (cs[i]->random != nullptr) return false;
        } else {
            auto it = idx.find(cs[i]->random);            // must land inside the copy
            if (it == idx.end() || it->second != rnd[i]) return false;
        }
    }
    return true;
}
//__USER__
int main() {
    {
        vector<int> vals{7,13,11,10,1}; vector<int> rnd{-1,0,4,2,0};
        Node* h = buildRand(vals, rnd);
        if (!checkCopy(h, copyRandomList(h), vals, rnd)) { std::puts("case1"); return 1; }
    }
    {
        vector<int> vals{1,2}; vector<int> rnd{1,1};
        Node* h = buildRand(vals, rnd);
        if (!checkCopy(h, copyRandomList(h), vals, rnd)) { std::puts("case2"); return 1; }
    }
    {
        vector<int> vals{}; vector<int> rnd{};
        Node* h = buildRand(vals, rnd);
        if (!checkCopy(h, copyRandomList(h), vals, rnd)) { std::puts("case3"); return 1; }
    }
    {
        vector<int> vals{3,3,3}; vector<int> rnd{-1,0,-1};
        Node* h = buildRand(vals, rnd);
        if (!checkCopy(h, copyRandomList(h), vals, rnd)) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The `random` links make a naive one-pass copy impossible because a target node may not exist yet. The O(1)-space technique splices each new node immediately after its original, so the copy of any node `x` is always `x->next`; that makes `x->random->next` the correct copied random. A final unweaving pass detaches the interleaved copies and restores the original list. O(n) time, O(1) auxiliary space (beyond the output), versus the simpler hash-map-of-clones alternative that costs O(n) space.
