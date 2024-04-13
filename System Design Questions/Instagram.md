### Problem Description
> Design Instagram

### Questions to ask


### Functional Requirements
1. User should be able to upload images and videos 
2. User should be able to follow other users
3. User should get a newsfeed on login 
4. User should be able to search for posts 
5. User should get notifications 
6. Capture analytics (number of likes / views) and comments 
7. Posts should be visible only to the people user has permissioned 

### Non Functional Requirements
1. System should be highly available 
2. System can be eventually consistent - SLA 5 minutes 
3. Durability - uploaded data should not be lost 
4. Content moderation 
5. Metrics collection 

### Capacity Estimation
1. 100 million users with 50 million DAU 
2. 20% users make posts, on an average 5 photos (5 Mb) and 1 video (150 Mb)
3. Write volume
	1. 10 million users * 6 files = 60 million write requests / day = 600 writes / sec. Peak volume = 1200 writes / sec
	2. 60 million * 200 Mb = 1 Petabyte / day = 400 Petabyte / year 
4. Read volume = 50 million * 20 requests = 1 billion requests / day = 10000 QPS. Peak QPS = 20000

### APIs
1. POST /postMedia
		postMedia(userId, mediaType, hashtags, caption)
2. POST /followUser , /unfollowUser
		followUser(userId, targetUserId)
		unfollowUser(userId, targetUserId)
3. POST /likePost , /dislikePost
		likePost(userId, postId)
		dislikePost(userId, postId)
4. GET /searchPhotos
		searchPhotos(userId, keyword)
5. GET /viewNewsFeed
		viewNewsFeed(userId)

### Object Model
1. Database tables:
	1. Users
	2. Photos
	3. UserFollowers

### HLD

![[Instagram Design.png]]


| Component        | Details                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Upload Service   | Uploads to S3 and updates metadata. Could update metadata initially with status pending, then mark status to uploaded when you get S3 event notification that the upload is complete                                                                                                                                                                                                                        |
| User Service     | 1. Helps with creation of user, adding follower<br>2. Keeps count of followers and marks celebrities when threshold is crossed                                                                                                                                                                                                                                                                              |
| Timeline Service | Hybrid approach for generation - reads celebrity status from user data (that is managed by User Service)                                                                                                                                                                                                                                                                                                    |
| Read Service     | 1. Gets timeline from Timeline Service<br>2. Keeps a count of views in local counters. If we have multiple instances then every fixed time interval, each service would publish its count on photoId to a queue and reset its counter. You can have a separate service reading counts from the queue, aggregating across multiple entries from the multiple instances and incrementing metadata accordingly |



### Database Choices
1. [[Amazon S3]] for storing photos and videos 
2. [[REDIS]] key-value store to store the timeline 
3. Relational database for user and content metadata 

### Interesting Algorithms/Approaches
1. Generating timeline / newsfeed - hybrid approach. Celebrities updates are **pulled** when user following them logs in. Normal users updates are **pushed** to their followers as and when they are made. Both are combined to generate timeline. 
2. Separate servers to handle reads and writes. We have a lot more reads than writes, write operations are more heavy and could interfere with reads. Best to separate them out and scale each independently. 

### Scaling Challenges
1. Sharding of data:
	1. Sharding based on userId - each shard has its own auto-incrementing sequence and appends shardId with this sequence to generate unique photoId
		1. Many celebrities on a given shard can cause imbalanced distribution of data 
		2. If a given shard is unavailable or slow, it will impact all data of the users on that shard
	2. Have auto-incrementing photoId (REDIS counter? / [[Unique ID Generator]]?) and shard based on that. Or have a key generation scheme - see URL shortening. 

### Deep Dive Topics
1. How to ensure metadata and S3 upload is in sync?
2. Which database for metadata and user data? How to shard data? 

### References

https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-instagram
https://www.educative.io/courses/grokking-the-system-design-interview/m2yDVZnQ8lG
