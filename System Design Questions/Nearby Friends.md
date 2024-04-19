### Problem Description


### Questions to ask
1. How geographically close is nearby?
2. Can we assume distance is straight line distance
3. How many users?
4. Do we need to store location history 
5. Do we need to show last known location of inactive friends

### Functional Requirements
1. User should be able to see nearby friends
2. Nearby list should be updated every few seconds 

### Non Functional Requirements
1. Low latency
2. Reliability
3. Eventual consistency 

### Capacity Estimation
1. 10 million DAU, 10% are concurrent 
2. Location update every 30 secs -> 10 million / 30 = 300k QPS
3. if a user has 500 friends and 10% are online -> 15 million location updates/sec

### APIs
1. Periodic location update -> client sends latitude/longitude/timestamp -> no response
2. Client receives location update -> friend location data and timestamp
3. Websocket initialization -> client sends latitude/longitude/timestamp -> friend location data and timestamp
4. Subscribe to a new friend -> friend id -> friend location data and timestamp
5. Unsubscribe a friend -> friend id -> no response 

### Object Model


### HLD
![[Nearby Friends Design.png]]

| Component            | Detail                                                                                                                                                                                                                                                                                                                                                                           |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RESTful API servers  | HTTP servers that handle adding/removing friends, updating user profile, etc. Stateless and easy to scale                                                                                                                                                                                                                                                                        |
| Websocket servers    | Each client maintains a persistent websocket connection to one of these servers. A websocket connection handler for each user subscribes to the user's friends' channels. It reads from the pub/sub channels and calculates the distance. If it is within the search radius, it will update the user                                                                             |
| REDIS location cache | Stores most recent location for a user with TTL - useful during client initialization to fetch friends' locations. We only care about the current location of a user and it need not be durable. If the cache goes down, it can be initialized with empty data and get filled as new location updates stream in. Shard by userId and replicate data to reduce the impact further |
| REDIS PubSub server  | Channels are cheap to create, have a channel for every user, every friend of the user subscribes to his channel. See [[REDIS#Redis PubSub]]                                                                                                                                                                                                                                      |

### Database Choices
1. User database (storing user profile and friendship data) - could use relational or NoSQL database. Shard by userId 
2. Location history database - Cassandra - handles heavy write workloads and can be horizontally scaled. Shard by userId 

### Interesting Algorithms/Approaches
1. Client initialization - when the websocket connection is initialized and the client sends the initial location of the user to the server:
	1. Server updates user location in the cache
	2. Saves the location in a variable in the connection handler for subsequent calculations
	3. Loads user's friends from the database and makes a batch request to fetch locations of the friends from the cache. Updates the user depending on the distance 
	4. Subscribe to all friends' channels and send the user's current location to the user's channel 
2. Distributed Redis Pub/Sub cluster - while memory is not an issue, CPU could become a bottleneck given the large number of messages/sec. 
	1. We should shard by userId and use [[Consistent Hashing]]. The websocket server consults the hash ring to decide which PubSub server to publish/subscribe to. Service discovery like [[Zookeeper]] could hold the ring config, but it can also be cached on each websocket server. 
	2. A Pub/Sub server is stateful, since if a channel moves, all subscribers will need to move as well. With stateful clusters, scaling up/down has operational overhead and risks, so the cluster is normally overprovisioned to avoid unnecessary resizing 
	3. If we do need to resize, there will be a lot of resubscription requests and updates might be missed. It should be done when usage is lowest. 
	4. Operational risk of replacing a Pub/Sub server is much lower. Only the channels on the server being replaced need to be handled. Every websocket server has a list of channels it has subscribed to, and when it gets an update notification, it checks each channel against the hash ring to determine if the channel needs to be resubscribed to the new server. 
3. Adding/Removing friends - Register a callback whenever a new friend is added / removed 
4. To change design from "Nearby Friends" to "Nearby Random Person", the Pub/Sub queues can be by the geohash instead of userId. 

### Scaling Challenges
1. Scaling the websocket servers is difficult because they are stateful. Mark a node as "draining" and allow all existing connections to drain, while no new websocket connections are routed to it. Once all existing connections are closed, the server can be removed. A good load balancer should be able to handle this. 
2. Users with many friends - subscribers are scattered among the websocket servers, so the update load will be spread among them. The user will place a bit more load on the Pub/Sub server but these users are spread among the Pub/Sub servers and incremental load will not overwhelm any single one. 

### Deep Dive Topics


### References
System Design Interview An Insiders Guide Volume 2
