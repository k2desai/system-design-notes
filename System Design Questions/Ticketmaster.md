### Problem Description


### Questions to ask


### Functional Requirements
1. User should be able to search for events
2. User should be able to view events
3. User should be able to book tickets
4. Dynamic pricing - Out of scope
5. Admins should be able to add events - Out of scope

### Non Functional Requirements
1. High availability for searching and viewing events
2. High consistency for booking events
3. Scalability - handle high throughput like popular events
4. Read heavy - high read throughput

### Capacity Estimation


### APIs
1. GET /event/details/{eventId}
2. POST /booking/checkout
3. POST /booking/confirm
4. GET /event/search/keyword={keyword}

### Object Model
1. Event 
2. Performer
3. Venue
4. Ticket
5. Booking

### HLD

![[Ticketmaster Design.png]]

| Component             | Detail                                                                                                                                                                                                                                                                                                                                                |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Event CRUD Service    | Reading / viewing events. Responsible for creating / updating / deleting events                                                                                                                                                                                                                                                                       |
| Ticket Lock (Redis)   | To ensure that the ticket is locked for a user while they are checking out. The lock is released if the user completes the purchase or if the TTL expires                                                                                                                                                                                             |
| Search Service        | Queries the events db for events matching the search parameters - this can be optimized by database indexing and query optimization, but it increases storage requirements and doesn't work well for partial matches. Use ElasticSearch (with CDC to ensure it is in sync with the database) to enable fuzzy search which allows for error tolerance. |
| Cache                 | Events details does not change frequently and can be cached with a read-through cache. Database triggers and TTL can invalidate cache entries                                                                                                                                                                                                         |
| Virtual waiting queue | For extremely popular events - it is possible that seat map goes stale frequently. Rather than use SSE to update the map (it might fill up immediately), put users in a queue before seeing the booking page using a websocket connection.                                                                                                            |
| CDN                   | To cache search results closer to the user                                                                                                                                                                                                                                                                                                            |

### Database Choices


### Interesting Algorithms/Approaches


### Scaling Challenges


### Deep Dive Topics


### References
https://www.hellointerview.com/learn/system-design/answer-keys/ticketmaster
https://www.educative.io/courses/grokking-the-system-design-interview/YQyq6mBKq4n
