When we hit our underlying database, we fetch extra results and cache them in Redis / Memcached with a long TTL. The timestamp of the last result tells us the next N results that we need to return. 