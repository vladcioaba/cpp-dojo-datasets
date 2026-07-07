## quiz: Exchanges send identical A and B multicast feeds. What does line-arbitration logic do?
tags: hft, feed-handler
track: fpga

- [ ] Load-balances outbound orders across two exchanges
- [x] Takes whichever copy of each sequence number arrives first and drops the duplicate, recovering packets lost on one feed
- [ ] Encrypts the market-data stream before parsing
- [ ] Doubles the rate at which orders can be sent

> A and B are redundant copies of the same feed. Arbitration deduplicates by sequence number and uses the earlier arrival, so a drop on one feed is covered by the other with no added latency. Doing it in the FPGA keeps recovery off the software critical path.
