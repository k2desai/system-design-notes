### Problem Description
Design a system like Yelp

### Questions to ask
1. Can the user define/change a search radius
2. What is the maximum search radius 
3. Should you expand the search if not enough results available 
4. How can business information get added/deleted/updated? Do we need realtime updates?
5. Can a user be moving while using the app?

### Functional Requirements
1. Return all businesses based on location
2. Business owners add/delete/update information
3. Customer can view detailed information about a business
4. Users should be able to add feedback about a place 

### Non Functional Requirements
1. Low latency
2. High availability and scalability
3. User location data privacy

### Capacity Estimation
100 million DAU, 5 queries per day -> 5000 QPS

### APIs
1. GET /v1/search/nearby?latitude=<>&longitude=<>&radius=<>
2. GET /v1/business/{id}
3. POST /v1/business
4. PUT /v1/business/{id}
5. DELETE /v1/business/{id}

### Object Model


### HLD
![[Proximity Service Design.png]]

### Database Choices
1. Relational DB because of high read:write ratio. Primary-secondary setup where primary handles writes and replicates to secondaries for reads 
2. Business data might be large and will need to be sharded 
3. [[Location algorithms#Geohashing]] table data (geohash and business id) is not too large and does not need sharding. Caching may not be needed, the dataset is relatively small and will fit in the working set of any database server - just add database read replicas to improve read performance. 

### Interesting Algorithms/Approaches
1. Deploy location based service to multiple regions and availability zones, making users physically closer to the systems. 

### Scaling Challenges
1. Both Business Service and Location Based Service are stateless, so easy to scale

### Deep Dive Topics
[[Location algorithms]]

### References
https://www.educative.io/courses/grokking-the-system-design-interview/B8rpM8E16LQ
System Design Interview An Insiders Guide Volume 2