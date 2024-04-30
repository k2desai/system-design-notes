### Redis PubSub
1. Very lightweight to create new channels 
2. A new channel is created when someone subscribes to it
3. If a message is published to a channel with no subscribers, the message is dropped, placing very little load on the server 
4. When a channel is created, a small amount of memory is used to maintain a hashtable and linked list to track the subcribers. 
5. If there is no update on a channel when a user is offline, no CPU cycles are used after the channel is created 

### Redis Sorted Sets
1. A Redis sorted set is a collection of unique strings (members) ordered by an associated score.
2. Internally it is implemented with hash table and skip list. Hash table maps the elements to their scores and skip list maps scores to elements. 
3. ZADD, ZINCRBY, ZRANGE, ZREVRANGE, ZRANK, ZREVRANK