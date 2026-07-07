## quiz: You quote bid $49.90 / ask $50.10 on a stock whose fair value is exactly $50.00. All order flow is uninformed and hits your bid or ask with equal probability. What is your expected profit per trade?
tags: market-making, expected-value, spread
track: quant

- [x] $0.10
- [ ] $0.20
- [ ] $0.05
- [ ] $0.00

> With uninformed flow you capture the half-spread on every fill. If they sell to you, you buy at $49.90 (fair $50.00) → +$0.10; if they buy, you sell at $50.10 → +$0.10. Expected profit = ½·0.10 + ½·0.10 = $0.10, exactly half the $0.20 spread. The full spread $0.20 is earned only across a round-trip (one buy and one sell).
