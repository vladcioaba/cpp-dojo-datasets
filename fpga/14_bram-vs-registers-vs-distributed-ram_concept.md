## fact: BRAM vs registers vs distributed RAM
tags: memory, resources
track: fpga

Three ways to store data, each with a tradeoff. **Flip-flops/registers** hold small amounts of state you touch every cycle — fully parallel but expensive per bit. **Distributed RAM** repurposes LUTs into small memories, handy for tiny tables close to the logic. **BRAM** is dedicated dual-port memory for larger buffers, FIFOs, and lookup tables.

Rule of thumb: a few bits, always live → registers; a few hundred entries → distributed RAM; kilobits and up → BRAM. Pick wrong and you either waste scarce resources or create a bottleneck. BRAM is dual-port, so at most two independent accesses per cycle — need more and you must replicate or bank it.
