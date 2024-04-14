### Problem Description


### Questions to ask
1. Mobile app / web app / both?
2. What are the important features?
3. Is the feed sorted by reverse chronological order and or scores (close friends have higher score)
4. How many friends can a user have?
5. What is the traffic volume?
6. Can feed contain images / videos / text?

### Functional Requirements
1. User should be able to create posts 
2. User should be able to follow / friend people
3. User should be able to view a feed of posts from people they follow, in chronological order 
4. User should be able to page through their feed
5. User should be able to like / comment on posts - Out of scope

### Non Functional Requirements
1. Highly available
2. Eventually consistent - upto 2 minutes delay is acceptable 
3. Posting and viewing feed should be fast - return in <500ms
4. Low latency
5. Scalable 

### Capacity Estimation
1. 300 million DAU, each user fetching timeline 5 times/day -> 1.5billion request / day -> 15000 QPS

### APIs
1. POST /post/create
2. POST /user/{id}/follow
3. GET /feed 

### Object Model
1. User
2. Follow
3. Post

### HLD

![[Newsfeed Design.png]]


### Database Choices
1. We can use DynamoDB for follow relationship, we don't need a graph database like Neo4j (this is more useful if you want to perform complex operations like friends-of-friends or friend recommendations)

### Interesting Algorithms/Approaches
1. To create "infinite scroll" experience, we send a timestamp with the feed request, so that we will only return feeds older than that timestamp

### Scaling Challenges
1. We need to shard posts - could be on post id
2. We need to shard feed data - on user_id. Use [[Consistent Hashing]] for future growth and replication 
3. Can introduce blob storage for posts and [[CDN]] for performance 
4. Only hold post_id in the feed table and cache - then fetch content based on id

### Deep Dive Topics
1. "Fan-out" problem when a user is following a large number of people - for that you pregenerate and store the feed in the feed table
2. "Celebrity" problem when a user has a large number of followers - here the feed service will get their posts when follower logs in 
3. "Hot-key" problem for post table - some posts are read a lot more than others. Solved by introducing LRU cache which holds popular posts 
4. Can add a notification service to inform users that new content is available
5. We could have ranking service that has algorithms that sort the feed in something other than chronological order - number of likes / comments / shares / close friends 

### References

System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.hellointerview.com/learn/system-design/answer-keys/fb-news-feed
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-newsfeed-system
https://www.educative.io/courses/grokking-the-system-design-interview/gxpWJ3ZKYwl