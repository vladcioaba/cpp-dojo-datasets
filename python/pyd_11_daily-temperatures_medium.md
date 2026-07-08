## challenge: Daily Temperatures
tags: stack, array, monotonic-stack
track: python
lang: python
difficulty: medium

Given a list `temperatures`, return a list `answer` where `answer[i]` is the number of days you must wait after day `i` for a warmer temperature. If no warmer day ever comes, `answer[i]` is `0`.

Constraints: `1 <= len(temperatures) <= 10^5`, `30 <= temperatures[i] <= 100`.

Example: `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]` → `[1, 1, 4, 2, 1, 1, 0, 0]`. `temperatures = [30, 40, 50, 60]` → `[1, 1, 1, 0]`.

hint: For each day you want the next day to its right that is strictly warmer.
hint: A stack of "days still waiting for a warmer day" lets you resolve several at once.
hint: Keep the stack's temperatures decreasing; when today is warmer, pop and fill in the gaps.

```python
# starter
def daily_temperatures(temperatures):
    ...
```

```python
def daily_temperatures(temperatures):
    res = [0] * len(temperatures)
    stack = []  # indices of days awaiting a warmer day
    for i, t in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < t:
            j = stack.pop()
            res[j] = i - j
        stack.append(i)
    return res
```

```python
# harness
#__USER__
def _check():
    assert daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]) == [1, 1, 4, 2, 1, 1, 0, 0]
    assert daily_temperatures([30, 40, 50, 60]) == [1, 1, 1, 0]
    assert daily_temperatures([30, 60, 90]) == [1, 1, 0]
    assert daily_temperatures([90, 80, 70]) == [0, 0, 0]
    assert daily_temperatures([50]) == [0]
    assert daily_temperatures([]) == []
    print("PASS")

_check()
```

**Editorial:** Keep a stack of indices whose warmer day has not been found yet, with their temperatures decreasing from bottom to top. When the current day is warmer than the temperature at the top of the stack, that waiting day's answer is the index distance — pop it and record `i - j`, repeating while the top stays cooler. Each index is pushed and popped once, so O(n) time and O(n) space, versus an O(n²) scan-ahead.
