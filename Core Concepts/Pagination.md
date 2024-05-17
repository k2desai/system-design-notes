When we hit our underlying database, we fetch extra results and cache them in Redis / Memcached with a long TTL. The timestamp of the last result tells us the next N results that we need to return. 

https://dev.to/pragativerma18/unlocking-the-power-of-api-pagination-best-practices-and-strategies-4b49
https://nordicapis.com/how-to-paginate-a-web-api/
https://nordicapis.com/everything-you-need-to-know-about-api-pagination/