### Problem Description


### Questions to ask


### Functional Requirements
1. Create / edit / delete tweets
2. Follow other users 
3. Service should be able to create and display user timeline 
4. User can like or dislike tweets 
5. User can search 

### Non Functional Requirements
1. Handle high volume of tweets / likes / retweets
2. Service needs to be highly available 
3. Consistency can take a hit in the interest of availability
4. Low latency
5. Scalability and reliability 
6. Security and privacy of user data

### Capacity Estimation
1. 200 million DAU
2. 100 million tweets / day -> 100 million * 300 bytes -> 30 GB/day
3. Every 5th feed has a photo (200 KB), every 10th tweet has a video (2 MB) -> 24 TB/day
4. 1 user follows 200 other users  

### APIs


### Object Model


### HLD
![[Twitter Design.png]]

| Component          | Details                                                                                                                                                                                                                                                                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tweet CRUD Service | Handles create / edit / delete tweet, likes, retweets, tweet metadata. It needs to have high throughput. Explain both read and write path here                                                                                                                                                                                            |
| Cache              | To store frequently accessed tweets and speed up read access                                                                                                                                                                                                                                                                              |
| Reply CRUD Service | We separate replies from tweets for several reasons:<br>1. Scalability - popular tweets will have millions of replies - this ensures tweet document does not become too large and we can scale reply service independently<br>2. Performance - when you load a tweet you dont want to load all replies - this makes read fast for a tweet |
| ElasticSearch      | Build reverse index on tweet content, username and hashtags. CDC ensures changes are propagated to the elasticsearch.                                                                                                                                                                                                                     |
| Timeline fanout    | Builds timeline as content gets written. Each follower has its own timeline cache and the fanout prepends every new tweet to all followers' timeline caches                                                                                                                                                                               |

### Database Choices
1. [[Mongo DB]] is a good because supports fast reads and writes and we dont need complex joins
2. Relational database for user data - handles complex joins and aggregations for user analytics, allows for efficient querying on structured user attributes and ensures data integrity with ACID transactions

### Interesting Algorithms/Approaches
1. Tweet id is tweet creation time + autoincrementing sequence
2. Replies are stored separately but Reply Service does not have a separate read path because it does not make sense to read reply without tweet. The reply document db is indexed by tweet id so that every time a tweet is loaded, we can lookup the replies to that tweet id very fast. 

### Scaling Challenges


### Deep Dive Topics
1. [[Unique ID Generator]] for generating tweet ids - we then shard by this tweet id. 
2. [[News feed System]] for generating timeline 


### References
https://www.educative.io/courses/grokking-the-system-design-interview/m2G48X18NDO
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-twitter
https://www.youtube.com/watch?v=Nfa-uUHuFHg&ab_channel=HelloInterview-FAANGMockInterviews