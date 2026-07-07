## fact: HDL describes hardware — everything runs in parallel
tags: hdl, mindset
track: fpga

**Verilog** and **VHDL** are hardware *description* languages, not programming languages. When you write two `always` blocks or two `assign` statements they do not run one after another — they become two independent circuits that operate **simultaneously, every clock cycle**. There is no program counter and no "next line."

This is the hardest shift for a software engineer. Ordering in the source text is largely irrelevant; what matters is the data-flow graph you are describing. Concurrency isn't something you add — it is the default, and forcing work to happen *in sequence* (via a state machine) is the extra effort.
