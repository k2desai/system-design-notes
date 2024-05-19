### Problem Description


### Questions to ask


### Functional Requirements
1. Send message
2. Receive message
3. Create/delete queue
4. Delete message
5. Ordering guarantee (atleast once, atmost once, exactly once)

### Non Functional Requirements
1. Scalable 
2. Highly available
3. Highly performant
4. Durable

### Capacity Estimation


### APIs


### Object Model


### HLD
![[Distributed Message Queue Design.png]]

| Component        | Detail                                                                                                                                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend service | API Gateway                                                                                                                                                                                                      |
| Metadata service | Caching layer between frontend and persistent storage. High read:write ratio. Strong consistency preferred but not required. Consistent hashing ring based on queue                                              |
| Backend          | Cant use database because of high throughput requirements and high read:write ratio. So we use memory / file system. Frontend looks up metadata service to decide which backend host to forward the request to.  |


### Database Choices


### Interesting Algorithms/Approaches


### Scaling Challenges


### Deep Dive Topics
1. Each backend host can be a leader for a set of queues, with zookeeper managing this configuration (in-cluster manager). Other option is entire cluster is assigned to a queue so any host in cluster can handle requests for that queue (out-cluster manager). The managers manage further partitioning of very big queues. 
2. Queue creation - autocreation on first message. Delete queue should not be exposed via public APIs
3. Message deletion 
4. Message replication - sync vs async (durability vs latency)
5. Pull vs push for consumer
6. Monitoring - health and other metrics

### References
https://www.youtube.com/watch?v=iJLL-KPqBpM 
https://www.cloudamqp.com/blog/part1-rabbitmq-for-beginners-what-is-rabbitmq.html 