## quiz: On average, which pattern appears first when flipping a fair coin: HH or HT?
tags: expected-value, coin, markov
track: quant

- [ ] HH — expected 4 flips vs 6 for HT
- [x] HT — expected 4 flips vs 6 for HH
- [ ] They tie — both take 4 flips
- [ ] They tie — both take 6 flips

> E[HT] = 4 but E[HH] = 6. For HT, once you get an H you can never "lose ground": any later T finishes you, and extra H's keep you primed. For HH, a T after your first H throws you all the way back to zero. Solving HT: a = 1 + ½b + ½a and b = 1 + ½·0 + ½b give b = 2, a = 4.
