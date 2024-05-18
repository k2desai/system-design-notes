When we hit our underlying database, we fetch extra results and cache them in Redis / Memcached with a long TTL. The timestamp of the last result tells us the next N results that we need to return. 

https://dev.to/pragativerma18/unlocking-the-power-of-api-pagination-best-practices-and-strategies-4b49
https://nordicapis.com/how-to-paginate-a-web-api/
https://nordicapis.com/everything-you-need-to-know-about-api-pagination/

### Benefits:
1. Performance and resource usage - limits the dataset, reducing load on server / network / client
2. Enhanced user experience 
3. Scalability and flexibility 

### Techniques:
1. Offset-based pagination - specify page number and number of items per page 
	1. Straightforward to implement
	2. inefficient for large datasets - offset needs to be calculated and skipped 
2. Cursor based pagination - uses unique id or token to mark the position in the dataset 
	1. More efficient when dealing with rapidly changing data 
3. Time based pagination - specify start and end times 
4. Keyset pagination - relies on sorting and using unique attribute to determine starting point

### Other considerations:
1. Determining appropriate page size:
	1. For smaller records, we can accommodate larger page size 
	2. Consider network latency and bandwidth 
	3. Larger page size increases response time - strike a balance between page size and performance 
2. Can implement caching to store paginated data - could do time based expiration 