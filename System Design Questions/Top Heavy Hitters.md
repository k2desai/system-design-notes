### Problem Description
1. Most searched words on Google
2. Most played videos on YouTube
3. Most played songs on Spotify
4. Most shared posts on Facebook
5. Most liked photos on Instagram 

### Questions to ask


### Functional Requirements
1. Top K for a given time interval 

### Non Functional Requirements
1. Scalable 
2. Highly available
3. Highly performant 
4. Accurate

### Capacity Estimation


### APIs


### Object Model


### HLD

![[Top Heavy Hitters Design.png]]

Data processing pipeline gradually decreases the request rate

| Component                    | Details                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API Gateway                  | Entry point - this can do some initial aggregation before sending to the queue. We could implement a background process that reads data from the logs and does some initial aggregation. This data could be flushed to the queue when the buffer is full or at a certain time interval. We could do aggregation on a separate process depending on memory / CPU of the host. Or we could even skip aggregation altogether                                                                                       |
| Distributed Messaging System | Initially aggregated data is sent to Kafka where it is partitioned by key.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Fast Path                    | Approximate and fast calculation - creates count min sketch and aggregates the data. No need to partition since memory is not a problem - any message from Kafka can be processed by any Fast Processor host. We skip replication here, since this is approximate anyway so we can tolerate some data loss if host goes down. The slow processor will product accurate results anyway. Results need to be flushed to Storage every few seconds for fast results - store Top K list for any given time interval. |
| Slow Path                    | Precise and slow calculation - here we can dump data to HDFS file storage and apply Map Reduce to it. (Two Map Reduce - Frequency count and Top K)                                                                                                                                                                                                                                                                                                                                                              |
| Data Partitioner             | Reads batches of events and parses them into individual events. Each partition stores subset of events, so Partition Processor reads from each partition and aggregates it further. Partition Processor can also pass data to Storage.                                                                                                                                                                                                                                                                          |
| Storage                      | Store Top K for different time intervals. If exact time interval that is requested is not available, we will have to add up whatever is available and give back and approximate answer.                                                                                                                                                                                                                                                                                                                         |


### Database Choices


### Interesting Algorithms/Approaches
1. We could partition by key, get Top K for each partition and then merge across all partitions to get final Top K. But we need to redo the whole algorithm for every required time interval. We cant reuse the results of smaller time intervals to get the result of a larger time interval. 
2. Count-min sketch - here data size is fixed, if we use hashtable instead, then size can grow by a lot. 
3. Other Counter based algorithms - Lossy Counting, Space Saving, Sticky Sampling
4. Lambda Architecture - an approach to building stream processing applications on top of MapReduce and stream processing engines. We send events to a batch system and stream processing system in parallel and stitch together results from both systems at query time. Drawbacks - complexity. 

### Scaling Challenges


### Deep Dive Topics


### References
https://www.youtube.com/watch?v=kx-XDoPjoHw
https://www.hellointerview.com/learn/system-design/answer-keys/top-k 