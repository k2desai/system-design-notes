### Problem Description
A unique id helps to identify the flow of an event in the logs across multiple microservices. 

### Questions to ask
1. What are the characteristics of unique ids 
2. Does id need to increment by 1
3. Do ids only contain numerical values
4. What is id length requirement 
5. What is the scale of the system

### Functional Requirements
1. IDs must be unique 
2. ID must be numerical values only 
3. ID must fit into 64 bits 
4. IDs are ordered by date
5. Ability to generate over 10000 unique ids / sec 

### Non Functional Requirements
1. Scalability - can generate atleast 1 billion unique ids / day
2. Availability 

### Capacity Estimation


### APIs


### Object Model


### HLD


### Database Choices


### Interesting Algorithms/Approaches
1. Ticket server - centralized auto-increment feature in a single database server. It does not scale well and is a single point of failure. 
2. Multi-master replication - use database's auto increment feature, but increment by k where k is the number of database servers. However, it is hard to scale with multiple datacenters and id does not necessarily go up with time. It does not work well if a database server is added / removed
3. UUID - 128 bit unique number - very low probability of collision. It is a simple approach and easy to scale, but the id is too long and not increasing with time.
4. Twitter snowflake approach - Timestamp + Datacenter ID + machine ID + sequence number - 12 bit sequence number is reset every millisecond. Datacenter and machine id is chosen at startup time, any change requires careful review. We get 2^12 -> 4096 combinations per millisecond. 

### Scaling Challenges


### Deep Dive Topics


### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-sequencer
