## quiz: In a pipelined loop, an initiation interval (II) of 1 means:
tags: pipelining, fundamentals
track: fpga

- [ ] The loop body executes exactly once
- [ ] Each result has a total latency of one cycle
- [x] The pipeline accepts a new input every clock cycle
- [ ] The clock is divided by one

> II is the number of cycles between accepting consecutive inputs. II=1 is ideal throughput — one new item per cycle — regardless of how many cycles of latency each item needs to traverse the whole pipeline. II>1 means the pipeline stalls between inputs.
