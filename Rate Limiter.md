### Problem Description
Design a rate limiter to prevent server from being overloaded and prevent resource starvation caused by DOS attacks

### Questions to ask
1. Client side or server side -> server side
	1. Client side can be unreliable and can be forged by malicious actors. We don't have control over client implementation
	2. Instead of putting on server, we can put in the middleware, as part of API gateway 
	3. We have to make a decision based on current technology stack, whether you are already using API gateway, the rate limiting algorithm you want to use, etc 
2. Throttling based on user id, IP or something else -> flexible
	1. IP based could be an issue when multiple users share same IP
	2. User id based needs user to be authenticated correctly, else someone else could block a user if there is rate limiting on the login API
3. What is the scale of the system? Startup or big company with large user base?
4. Will it have to work in a distributed environment -> Yes
5. Separate service or in application code -> our decision
6. Do we need to inform users who are throttled -> Yes
7. Do we need strict throttling or some allowance is alright? -> This will decide which algorithm we use - hard limit vs soft limits (we also have dynamic throttling based on request availability)

### Functional Requirements
1. Accurately limit excessive requests
2. Show clear exceptions to users when throttled
3. Distributed rate limiting - can be shared across multiple servers / processes

### Non Functional Requirements
1. Low Latency - should not slow down request processing
2. Low memory overhead
3. Highly available 
4. High fault tolerance - any issues with rate limiter should not affect the entire system

### Capacity Estimation


### APIs


### Object Model
1. Counter - key / value
	key - combination of UserId / IP + API 
	value - count
2. Rules - API - count allowed per time unit

### HLD

![[Pasted image 20240412213701.png]]

1. Choosing middleware for rate limiting
2. Rules are stored on disk, worker pulls rule from disk and stores them on cache
3. Flow of operations:
	1. Client sends request to rate limiting middleware
	2. Middleware loads rules from cache and checks if limit is reached or not depending on userId and API by fetching the counter from the corresponding bucket. TODO - which algorithm?
	3. If limit is reached, request is rejected - send back 429 Too Many Requests error
	4. If limit is not reached, request is forwarded to API servers and system increments the counter and saves it back to Redis

### Database Choices
1. Where to store counter:
	1. Database will be too slow 
	2. Redis in-memory cache is faster and supports time-based expiration. It offers INCR and EXPIRE
	3. Shard based on user id, use consistent hashing 
	4. Use write back cache, update counters in memory and write to disk can be done at fixed intervals 

### Interesting Algorithms/Approaches
1. Token bucket
	1. Bucket size and refill rate - every request takes a token from bucket if available, else it is rejected
	2. Different buckets for different API endpoints - eg. making post, adding friend, liking a post. If throttling based on IP, then different bucket for each IP 
	3. Easy to implement and memory efficient
2. Leaking bucket
	1. Queue size and outflow rate - requests are pulled from queue at specified rate, request added to queue if capacity available
	2. Memory efficient
3. Fixed window counter
	1. Divides timeline into fixed windows, assigns a counter to each window. 
	2. Burst of traffic at edge of windows can cause more than allowed quota to go through
	3. Memory efficient and easy to understand
4. Sliding window log
	1. Request keeps a log of timestamp and removes all outdated timestamps before checking if capacity is available
	2. Very accurate
	3. Consumes a lot of memory 
5. Sliding window counter
	1. Combines #3 and #4 above, adds weighted count from previous window to current window. Smooths any spike in traffic

### Scaling Challenges


### Deep Dive Topics
1. Where are the rules stored? - Rules are written in configuration files and saved on disk 
2. Rate limiting errors - In case a request is rate limited, APIs return a HTTP response code 429 (too many requests) to the client. We will return following headers to client:
	1. Ratelimit-remaining 
	2. Ratelimit-limit
	3. Ratelimit-Retry-After
3. Rate limiter in distributed environment:
	1. Race condition - when two clients try to update the same value - can use locks but that will slow down the system. Can also consider set-then-get approach
	2. Synchronization - if we use multiple instances of rate limiter then need to makes sure data is synchronized across them - both used centralized data store like Redis to store the count 
4. Monitoring - gather analytics to see if our algorithm and rules are effective
5. Design clients to have exponential backoff in case of rate limiting
6. For optimizing performance, when client sends request, we just check count and forward request if it is allowed. The updation of the count and cache can happen offline
7. We did rate limiting at application layer, could do at other layers 

### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-the-system-design-interview/3jYKmrVAPGQ
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-the-rate-limiter