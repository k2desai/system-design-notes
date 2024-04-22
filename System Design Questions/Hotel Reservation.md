### Problem Description


### Questions to ask
1. What is the scale of the system?
2. Do customers pay in full at time of reservation?
3. Do customers book through hotel website only 
4. Can customers cancel their reservation?
5. Is there anything else we need to consider?

### Functional Requirements
1. Show hotel related page
2. Show hotel room related detail page
3. Reserve a room 
4. Admin panel to add/remove/udpate room
5. Support overbooking feature
6. Dynamic pricing

### Non Functional Requirements
1. Support high concurrency 
2. Moderate latency for booking request

### Capacity Estimation
1. 5000 hotels, 1 million total rooms 
2. 70% occupancy, 3 days average stay -> 1 million * 0.7 / 3 daily reservations -> 3 reservations/sec 
3. 300 QPS for viewing room details, 30 QPS for order booking page
4. 5000 hotels, 20 types of rooms, 2 years of data -> 80 million rows

### APIs
1. GET / POST / PUT / DELETE - /v1/hotels/ID
2. GET / POST / PUT / DELETE - /v1/hotels/ID/rooms/ID
3. GET / POST / PUT / DELETE - /v1/reservations/ID
4. 

### Object Model


### HLD

![[Hotel Reservation Design.png]]

### Database Choices
Relational DB because:
1. Read-heavy workflow
2. ACID transactions supported 
3. Easy to model data in relational DB
4. Data is not a lot, so can fit on a relational DB, however, it can becomes a single point of failure, so we setup database replication across multiple regions / availability zones 
5. You could move out historical data to make more space 
6. If you have to shard - hotel id could be a good sharding key
7. For performance, you can add a cache on top of the database - this is useful since 90% operations are read-only. The cache needs to be updated only when a reservation is actually made. 

### Interesting Algorithms/Approaches
1. Concurrency issues:
	1. If user clicks book button multiple times, you could either gray out the button on client side, or else have an idempotent API that uses reservation id to avoid double reservation. 
	2. If more bookings than the number of rooms available:
		1. Pessimistic locking - only one user can access a record, can cause deadlocks and has a performance impact -> not scalable
		2. Optimistic locking - validation while writing to database - performance drops when concurrency is high with lots of failures/retries
		3. Could have status and expiration time in database 

### Scaling Challenges
1. All services are stateless and can easily be scaled out 

### Deep Dive Topics
For managing data across microservices: [[Database Transactions]]

### References
System Design Interview An Insiders Guide Volume 2 