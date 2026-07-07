## fact: HLS — writing C++ that becomes hardware
tags: hls, workflow
track: fpga

**High-Level Synthesis** lets you write C or C++ and have the tool generate RTL. You annotate the code with pragmas to steer the hardware: `#pragma HLS pipeline` to pipeline a loop toward II=1, `#pragma HLS unroll` to replicate a loop body, `#pragma HLS array_partition` to split an array across BRAMs for parallel access.

```cpp
void mac(const int a[8], const int b[8], int& acc) {
#pragma HLS pipeline II=1
    int sum = 0;
    for (int i = 0; i < 8; ++i) {
#pragma HLS unroll
        sum += a[i] * b[i];   // 8 parallel multiplies
    }
    acc = sum;
}
```

HLS shortens development and lets software engineers contribute, but it hides the hardware: for the tightest latency hand-written RTL usually still wins, and you must understand the generated circuit to hit II=1 and close timing.
