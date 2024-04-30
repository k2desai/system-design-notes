### Problem Description


### Questions to ask
1. How is the score calculated?
2. Are all players included in the leaderboard?
3. Is there a time segment associated with the leaderboard?
4. Can we assume we only care about top 10 players?
5. How many users in a tournament
6. How many matches in a tournament
7. How do we determine rank in case of a tie
8. Does the leaderboard need to be realtime

### Functional Requirements
1. Display top 10 players 
2. Show a user's specific rank 
3. Display users 4 places above and below desired users - Bonus

### Non Functional Requirements
1. Realtime update on scores
2. Scalability, availability, reliability 
3. Historical scores - Bonus
4. Multiple games - Bonus 

### Capacity Estimation
1. 5 million DAU, 50 QPS -> Peak 250 QPS
2. 50 QPS for fetching leaderboard
3. 25 million monthly users -> 24 char id and 16 bit (2 byte) score -> 26 bytes / user -> 650 MB ( * 2 because we have hash table and skip list) - very easy to hold in a single REDIS server

### APIs
1. POST /v1/score
2. GET /v1/scores
3. GET /v1/score/{user_id}

### Object Model


### HLD
![[Leaderboard Design.png]]

| Component           | Detail                                                                                                      |
| ------------------- | ----------------------------------------------------------------------------------------------------------- |
| Gaming Service      | Determines score and then writes to score queue.                                                            |
| Leaderboard Service | To read leaderboard and rank. No user should call Leaderboard service directly for writes, only for reads.  |



### Database Choices
1. SQL database is not a good choice because they are not performant when we have to process a large amount of continuously changing information. We could store user meta data here
2. [[REDIS#Redis Sorted Sets]] is ideal for solving leaderboard problems. A single servers can easily support the memory requirements and 2500 QPS. For reliability, while redis does support persistence, restarting from disk is slow. Configure with a read replica instead. 
3. Influx timeseries database to store historical scores of user (every minute, etc). It would support aggregations and historical queries
4. DynamoDB / Cassandra / MongoDB - optimized for writes and can efficiently sort items within a partition. If we use DynamoDB, with denormalized data and secondary indices to access data by attribute other than primary key. Need to decide number of partitions carefully - more partitions reduces load on each partition but adds complexity in "scatter gather"

### Interesting Algorithms/Approaches
1. You could leverage cloud infrastructure - Amazon API Gateway to connect to AWS Lambda Functions - Lambda serverless functions run when needed and scale depending on traffic. AWS has support for Redis clients that can be called from Lambda functions. 

### Scaling Challenges
1. If users grow by 100 times, we need sharding. 
	1. Fixed partition sharding by score ranges - we may need to adjust the score range in each partition to make sure there is even distribution across all shards. We have to shard the data ourselves using application code. Also need to be careful about moving user between shards. But then fetching top scorers or rank becomes easy. 
	2. Redis cluster which shards by hash of key (user id). Getting leaderboard is difficult because you need to get Top 10 from each shard and then have the application sort amongst that. If k or number of partitions is larger, latency increases. No good way to get the rank of a user. 

### Deep Dive Topics


### References
System Design Interview An Insiders Guide Volume 2 