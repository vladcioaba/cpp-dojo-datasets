## fact: Latency vs throughput and the initiation interval
tags: pipelining, fundamentals
track: fpga

**Latency** is how long one item takes end to end; **throughput** is how many items complete per unit time. Pipelining trades a little latency for a lot of throughput. In HFT you care about both, but tick-to-trade is fundamentally a **latency** game — one message in, one order out, as fast as possible.

The **initiation interval (II)** is the number of clock cycles between accepting consecutive inputs. **II = 1** means the pipeline swallows a new input every cycle — the ideal. II > 1 means the hardware stalls between inputs, usually because of a resource or dependency that can't be shared fast enough.
