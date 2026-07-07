## fact: Parsing market data and A/B line arbitration in hardware
tags: hft, feed-handler, protocols
track: fpga

Exchange feeds are usually **fixed-layout binary** messages (many use FIX/FAST or SBE-style encodings). Fixed fields at known offsets are ideal for hardware: a pipeline pulls price, size, and sequence number out of each message with combinational field extraction, a new message per clock.

Exchanges also send two identical multicast feeds, **A and B**, for redundancy. **Line arbitration** logic takes whichever copy of each sequence number arrives first and drops the duplicate, so a packet lost on one feed is recovered from the other with no added latency. Doing this in the FPGA keeps recovery entirely off the software critical path.
