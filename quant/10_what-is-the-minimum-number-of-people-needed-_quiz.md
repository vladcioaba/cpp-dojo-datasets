## quiz: What is the minimum number of people needed for a better-than-even chance that two share a birthday (365 equally likely days)?
tags: combinatorics, birthday
track: quant

- [x] 23
- [ ] 183
- [ ] 366
- [ ] 30

> Compare pairs, not days. P(all distinct) with n people = 365/365 · 364/365 · … · (365−n+1)/365. This drops below 1/2 first at n = 23 (P(all distinct) ≈ 0.493, so P(match) ≈ 0.507). 183 is the "half of 365" trap; 366 only guarantees a match (pigeonhole).
