## challenge: Kth Largest Element in an Array
tags: heap, sorting, quickselect, divide-and-conquer
track: python
lang: python
difficulty: medium

Given a list `nums` and an integer `k`, return the `k`-th largest element in the array. This is the element that would sit at position `k` from the end if the array were sorted, not necessarily a distinct value.

Constraints: `1 <= k <= len(nums) <= 10^5`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [3, 2, 1, 5, 6, 4], k = 2` → `5`.

hint: Sorting works but does more than you need — you only care about the top `k`.
hint: Keep a min-heap of the `k` largest values seen so far.
hint: When the heap exceeds size `k`, pop the smallest; its root is then the answer.

```python
# starter
def find_kth_largest(nums, k):
    ...
```

```python
def find_kth_largest(nums, k):
    import heapq
    heap = []
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]
```

```python
# harness
#__USER__
def _check():
    assert find_kth_largest([3, 2, 1, 5, 6, 4], 2) == 5
    assert find_kth_largest([3, 2, 3, 1, 2, 4, 5, 5, 6], 4) == 4
    assert find_kth_largest([1], 1) == 1
    assert find_kth_largest([7, 6, 5, 4, 3, 2, 1], 5) == 3
    assert find_kth_largest([2, 1], 2) == 1
    print("PASS")

_check()
```

**Editorial:** Maintain a size-`k` min-heap of the largest elements seen. Each push is O(log k), and whenever the heap grows past `k` you evict its smallest root, so the heap always holds the top `k`. After the scan the root is the `k`-th largest. O(n log k) time, O(k) space — cheaper than a full O(n log n) sort.
