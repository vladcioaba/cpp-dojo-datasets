## quiz: What does `#pragma HLS pipeline II=1` request in this HLS loop?
tags: hls, workflow
track: fpga

```cpp
for (int i = 0; i < N; ++i) {
#pragma HLS pipeline II=1
    out[i] = f(in[i]);
}
```

- [ ] Run the loop on the host CPU instead of the FPGA
- [x] Generate hardware that starts a new loop iteration every clock cycle
- [ ] Replicate the loop body into N fully parallel copies
- [ ] Store `out` in floating-point format

> `pipeline II=1` asks HLS to build a pipelined datapath that accepts a new iteration each cycle. Full spatial replication is `#pragma HLS unroll` — a different transformation (though the two can combine). The pragma says nothing about numeric type or where the code runs.
