## fact: The toolchain — synthesis, place-and-route, bitstream
tags: toolchain, workflow
track: fpga

Getting from HDL to a running chip is a pipeline of tools. **Synthesis** turns your Verilog into a netlist of LUTs, flops, and hardened blocks. **Place-and-route** decides which physical LUT each piece lands in and threads the routing between them. **Static timing analysis** then checks every path against the clock. Finally a **bitstream** is generated and loaded onto the device.

The dominant tools are **AMD/Xilinx Vivado** and **Intel/Altera Quartus**. The pain point: builds are slow — place-and-route on a large design can take **hours**, and a one-line change means another full run. The edit-compile-test loop is nothing like software's.
