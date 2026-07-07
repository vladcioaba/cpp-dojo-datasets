## quiz: You have $30, your opponent $70, fair $1 bets until someone is ruined. What is the expected number of bets until the game ends?
tags: gamblers-ruin, random-walk, expected-value
track: quant

- [ ] 100
- [ ] 210
- [ ] 1050
- [x] 2100

> For a symmetric random walk with absorbing barriers at 0 and N, starting from k, the expected number of steps is k·(N−k). Here k=30, N=100, so E = 30·70 = 2100. The variance-like product makes the game surprisingly long even though each step is tiny.
