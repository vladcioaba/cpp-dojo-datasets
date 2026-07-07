## fact: The HFT FPGA ecosystem
tags: hft, vendors
track: fpga

The dominant FPGA vendors are **AMD/Xilinx** (UltraScale+ family, Alveo accelerator cards) and **Intel/Altera** (Stratix, Agilex). Low-latency **NICs** historically came from **Solarflare** (now part of AMD) and are common both for kernel bypass and as FPGA-NIC platforms. On the network side, **Arista** — including the former Exablaze/Metamako lines — builds ultra-low-latency switches, some with an FPGA layer for inline logic.

Toolwise you'll live in **Vivado** (Xilinx) or **Quartus** (Intel). Exact latency figures are vendor- and design-specific and worth measuring yourself rather than trusting a datasheet headline — but the vendors above are what most HFT hardware is built on.
