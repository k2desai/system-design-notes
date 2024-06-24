### Problem Description


### Questions to ask


### Functional Requirements


### Non Functional Requirements


### Capacity Estimation


### APIs


### Object Model


### HLD
![[Chess Design.png]]


### Database Choices


### Interesting Algorithms/Approaches


### Scaling Challenges


### Deep Dive Topics
1. Scatter gather in game db if i want to lookup all games for a user, avoid it by having user to game mapping 
2. Game move service is partitioned by game id
3. Scores in REDIS if i may want rank of any particular user, if only top k ranks, then use a heap


### References