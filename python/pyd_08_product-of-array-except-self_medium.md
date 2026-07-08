## challenge: Product of Array Except Self
tags: array, prefix-product
track: python
lang: python
difficulty: medium

Given a list `nums`, return a list `res` where `res[i]` is the product of every element except `nums[i]`. Solve it without using division.

Constraints: `1 <= len(nums) <= 10^5`, the full answer fits in a 32-bit integer. Runtime should be O(n).

Example: `nums = [1, 2, 3, 4]` → `[24, 12, 8, 6]`. `nums = [-1, 1, 0, -3, 3]` → `[0, 0, 9, 0, 0]`.

hint: Each answer is (product of everything to the left) times (product of everything to the right).
hint: Two sweeps — one left-to-right for prefixes, one right-to-left for suffixes.
hint: Fill the output with prefix products first, then multiply in the suffix products on the way back.

```python
# starter
def product_except_self(nums):
    ...
```

```python
def product_except_self(nums):
    n = len(nums)
    res = [1] * n
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]
    suffix = 1
    for i in range(n - 1, -1, -1):
        res[i] *= suffix
        suffix *= nums[i]
    return res
```

```python
# harness
#__USER__
def _check():
    assert product_except_self([1, 2, 3, 4]) == [24, 12, 8, 6]
    assert product_except_self([-1, 1, 0, -3, 3]) == [0, 0, 9, 0, 0]
    assert product_except_self([2, 3]) == [3, 2]
    assert product_except_self([5]) == [1]
    assert product_except_self([0, 0]) == [0, 0]
    assert product_except_self([1, 1, 1]) == [1, 1, 1]
    print("PASS")

_check()
```

**Editorial:** The product excluding index `i` is the product of all elements before `i` times the product of all elements after `i`. First pass left-to-right writes the running prefix product into `res[i]` (before multiplying `nums[i]` in). Second pass right-to-left multiplies each `res[i]` by the running suffix product. This handles zeros naturally and uses only the output array, giving O(n) time and O(1) extra space.
