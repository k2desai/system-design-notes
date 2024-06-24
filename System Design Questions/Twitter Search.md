### Problem Description


### Questions to ask


### Functional Requirements
1. User should be able to search for tweets by keyword
2. User should be able to sort search results by popularity or chronologically
3. Users should be able to access results in pages 
4. Fuzzy / Personalized search - Out of scope

### Non Functional Requirements
1. Fast, median queries return in <500ms
2. Support high volume of requests
3. Highly available 
4. Scalable

### Capacity Estimation
1. 100 million users, average 1 tweet/day and 10 likes/day -> 1k tweets/second, 10k likes/second. Likes is significantly more than tweet creation. 
2. 1k search/second -> system is write heavy
3. 1 tweet -> 1 KB, 10 years data -> 365 B tweets -> 365 TB

### APIs
1. GET /search?query=THE_QUERY&sort_order=LIKES|TIME

### Object Model


### HLD
![[Twitter Search Design.png]]

![[Search Design.png]]

| Component         | Detail                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| ElasticSearch     | Powerful search service based on Lucene which supports scaling, replication, creating arbitrary full-text indices on your data |
| Search Cache      | Use for [[Pagination]]                                                                                                         |
| Write path        | Ingestion service to cut down on write volume.                                                                                 |
| Query path        | Two layers of caching - Request level and in front of ElasticSearch.                                                           |
| Reranking Service | Grabs up-to-date counts before returning to the user as what is written to ElasticSearch may be stale.                         |

### Database Choices


### Interesting Algorithms/Approaches
1. You would need to batch the writes before writing them to the database. Could have an aggregator service to do this. You could only store write count when it passes milestones like power of 2/10, getting an approximate like count. 
2. We can create our own reverse search index where the key is the word and value is the list of tweet ids that hold that word. Partition the data based on word hash. The lookups can be stored in DynamoDB
3. We use document partitioning (makes writing of index easier). With term partitioning, when we have multi-word queries, then we would be searching across partitions in any case

### Scaling Challenges


### Deep Dive Topics


### References
https://www.hellointerview.com/learn/system-design/answer-keys/tweet-search
https://www.educative.io/courses/grokking-the-system-design-interview/xV9mMjj74gE
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-the-distributed-search