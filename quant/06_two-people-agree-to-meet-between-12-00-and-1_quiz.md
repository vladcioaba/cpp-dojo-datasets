## quiz: Two people agree to meet between 12:00 and 1:00, each arriving at a uniformly random time and waiting 15 minutes. What is the probability they actually meet?
tags: geometric-probability, continuous
track: quant

- [ ] 1/4
- [x] 7/16
- [ ] 9/16
- [ ] 1/2

> Scale the hour to [0,1]; 15 min = 1/4. They meet iff |x−y| ≤ 1/4. In the unit square the "miss" region is two right triangles with legs 3/4, total area 2·½·(3/4)² = (3/4)² = 9/16. So P(meet) = 1 − 9/16 = 7/16.
