## quiz: A family has two children. You learn at least one is a boy. What is the probability both are boys?
tags: conditional-probability, bayes
track: quant

- [ ] 1/2
- [x] 1/3
- [ ] 1/4
- [ ] 2/3

> Equally likely birth orders are {BB, BG, GB, GG}. Conditioning on "at least one boy" removes GG, leaving {BB, BG, GB}. Only BB has two boys, so the probability is 1/3. The tempting 1/2 forgets that BG and GB are two distinct outcomes.
