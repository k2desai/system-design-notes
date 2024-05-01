
#### Cache aside (Lazy Loading):
The application is responsible for reading and writing from storage. The cache does not interact with storage directly.
Disadvantages:
1. Each cache miss results in three trips, which can cause a noticeable delay.
2. Data can become stale if it is updated in the database. This issue is mitigated by setting a time-to-live (TTL) which forces an update of the cache entry, or by using write-through.
3. When a node fails, it is replaced by a new, empty node, increasing latency.

#### Read Through:
1. The cache checks if the data is available (cache hit).
2. If the data is not in the cache (cache miss), the cache system reads the data from the primary storage.
3. The cache stores the data in the cache.
4. The cache returns the data to the client.
5. Subsequent read requests for the same data will be served directly from the cache until the data expires or is evicted.

#### Write Back:
In write-behind, the application does the following:
1. Add/update entry in cache
2. Asynchronously write entry to the data store, improving write performance
Disadvantages:
1. There could be data loss if the cache goes down prior to its contents hitting the data store.
2. It is more complex to implement write-behind than it is to implement cache-aside or write-through.

#### Write Through:
The application uses the cache as the main data store, reading and writing data to it, while the cache is responsible for reading and writing to the database. 
Disadvantages:
1. When a new node is created due to failure or scaling, the new node will not cache entries until the entry is updated in the database. Cache-aside in conjunction with write through can mitigate this issue.
2. Most data written might never be read, which can be minimized with a TTL.

#### Write-around cache
This strategy involves writing data to the database only. Later, when a read is triggered for the data, it’s written to cache after a cache miss. The database will have updated data, but such a strategy isn’t favorable for reading recently updated data.

#### Refresh ahead
You can configure the cache to automatically refresh any recently accessed cache entry prior to its expiration.
### References
https://github.com/donnemartin/system-design-primer