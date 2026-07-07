## quiz: Same data, but one run sorts the array first — which loop is faster?
tags: branch-prediction, performance
track: hft

```cpp
// large array of random 0..255 values
long sum = 0;
for (int x : data)
    if (x >= 128) sum += x;
```

- [x] Sorted is much faster: the branch becomes predictable (a long not-taken run, then a long taken run), so the CPU rarely mispredicts and flushes the pipeline
- [ ] Unsorted is faster because sorting evicts the data from cache
- [ ] Identical — a single comparison is not affected by branch prediction
- [ ] Sorted is slower because the sort's O(n log n) cost is paid inside the loop

> On random data the `x >= 128` branch is taken about half the time and is essentially unpredictable, so the branch predictor misses often and each miss flushes the pipeline (tens of cycles). After sorting, the branch is not-taken for the whole first half and taken for the second half — trivially predictable — so misprediction nearly vanishes. Rewriting it branchlessly (e.g. `sum += (x >= 128) * x` / a `cmov`) removes the dependence on prediction entirely.
