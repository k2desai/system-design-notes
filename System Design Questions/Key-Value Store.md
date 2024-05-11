### Problem Description
> Problem Description

### Questions to ask


### Functional Requirements
1. get / put value for key 

### Non Functional Requirements
1. High availability
2. High scalability with automatic scaling
3. Tunable consistency
4. Low latency 
5. Fault tolerance 

### Capacity Estimation
1. Key-value pair is less than 10 KB
2. Ability to store big data

### APIs
1. get(key)
2. put(key, value)

### Object Model


### HLD
![[Images/Key-value Write Design.png]]
![[Key-value Read Design.png]]
![[Distributed Cache Design.png]]
### Database Choices


### Interesting Algorithms/Approaches
1. Every node is responsible for 
	1. Storage engine
	2. Client API
	3. Failure Detection - using [[Consistency#Failure Detection]]
	4. Failure repair - using [[Consistency#Handling Temporary Failures]] and  [[Consistency#Handling Permanent Failures]]
	5. Highly available writes - Conflict resolution - using vector clocks 
	6. Highly available reads - Replication - using [[Consistent Hashing]]
2. [[Bloom filters]] are used to find which SSTable contains the requested key
### Scaling Challenges


### Deep Dive Topics


### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-the-key-value-store
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-the-distributed-cache