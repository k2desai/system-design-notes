### Problem Description


### Questions to ask


### Functional Requirements
1. User should be able to post new tweets
2. User should be able to follow other users 
3. Service should be able to create and display user timeline 
4. User can like or dislike tweets 
5. User can search 

### Non Functional Requirements
1. Service needs to be highly available 
2. Consistency can take a hit in the interest of availability
3. Low latency
4. Scalability and reliability 

### Capacity Estimation
1. 200 million DAU
2. 100 million tweets / day -> 100 million * 300 bytes -> 30 GB/day
3. Every 5th feed has a photo (200 KB), every 10th tweet has a video (2 MB) -> 24 TB/day
4. 1 user follows 200 other users 
5. 

### APIs


### Object Model


### HLD
![[Twitter Design.png]]

### Database Choices


### Interesting Algorithms/Approaches
1. Tweet id is tweet creation time + autoincrementing sequence

### Scaling Challenges


### Deep Dive Topics
1. [[Unique ID Generator]] for generating tweet ids - we then shard by this tweet id. 
2. [[News feed System]] for generating timeline 


### References
https://www.educative.io/courses/grokking-the-system-design-interview/m2G48X18NDO
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-twitter
