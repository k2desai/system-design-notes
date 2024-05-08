### Problem Description
> Problem Description

### Questions to ask


### Functional Requirements
1. Riders should be able to input source and destination to see an estimated fare
2. Riders should be able to request a ride and be matched with a driver in real time
3. Drivers should be able to accept / reject ride requests
4. Rating riders / drivers - out of scope
5. Different categories of rides - out of scope

### Non Functional Requirements
1. Low latency ride matching 
2. Strong consistency in ride matching 
3. Highly available and reliable 
4. High throughput during peak hours / events 

### Capacity Estimation


### APIs
1. POST /fare-estimate?pickup={location}&destination={location} -> Partial
```
Partial<Ride>: 
{
	"id": number, 
	"price": number, 
	"eta": DateTime 
}
```
2. PATCH /rides/request
```
Body:
    {
        "rideId": string, // This is the id returned from the fare estimate endpoint
    }
```
3. POST /drivers/location/update
4. PATCH /rides/accept
5. PATCH /rides/status/update
### Object Model
1. Rider
2. Driver
3. Ride
4. Location

### HLD

![[Uber Design.png]]

| Component               | Detail                                                                                                                                                                                                      |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ride Service            | Manages ride state starting with calculating fare estimate till trip end                                                                                                                                    |
| Third party mapping API | Google maps - provide routing functionality                                                                                                                                                                 |
| Location service        | Manages real-time location of drivers                                                                                                                                                                       |
| Notification service    | To notify drivers when new rides are matched to them                                                                                                                                                        |
| Distributed lock        | To prevent multiple ride requests being sent to the same driver - use a distributed lock to lock the driver id with a TTL. Since locks are held for upto 10 seconds, its easier to implement failover logic |
| Queue                   | To ensure no rides are dropped - Ride matching service needs to scale depending on size of queue                                                                                                            |


### Database Choices
1. Cassandra for location information? 
2. Relational db for ride information? 

### Interesting Algorithms/Approaches
1. Geosharding with read replicas - sharding our data geographically allows us to scale and reduces latency by reducing distance between client and server. The only time we need to scatter-gather (query multiple shards) is when we are doing proximity search on a boundary

### Scaling Challenges
1. To handle large volumes of driver location data - use Redis which can support geospatial data types and lets us handle location update and proximity search with low latency. Can set TTL to expire old data. Make sure to monitor memory and scale as needed
2. To handle frequent location updates - logic on client side to dynamically adjust number of location updates to send based on driver speed, direction of travel, etc 

### Deep Dive Topics
[[Location algorithms#Quadtrees]] - updating a quadtree is time-consuming, so we use a hashtable to store driver's latest location and update quadtree every 15 secs 
[[Location algorithms#Geohashing]]
To find shortest path - Dijakstra's algorithm and [[Location algorithms#Contraction hierarchy]]

### Drilldown:
1. Driver location update - adaptive 
2. Store data keyed by geohash (special geohash cells of different sizes)
3. If many riders querying a ride - use a queue - to manage booking service going down, and locking for not sending multiple requests to same driver 

### References
1. https://www.hellointerview.com/learn/system-design/answer-keys/uber
2. https://www.educative.io/courses/grokking-the-system-design-interview/YQVkjp548NM
3. https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-uber