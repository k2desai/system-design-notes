If we use hash%N for partitioning, it wont work well if servers are added / removed. Instead SHA-1 is used as hash function and the hash space goes from 0 - 2^160-1. We use the same hash function to map servers on the ring (based on IP/server name) as well as the keys. To determine which server a key is stored on, go clockwise from the key until a server is found. Adding / removing server only requires redistribution of a fraction of keys - we need to move anti-clockwise until we find a server to find affected keys. 

To avoid non-uniform key distribution and to keep partitions roughly the same size, we use virtual nodes. As the number of virtual nodes increases, the more balanced the ring becomes. Typically we use 100-200 virtual nodes - we can tune this as per system requirements

Advantages:
1. Minimized keys redistributed when servers added / removed
2. Easy to scale horizontally because data is more evenly distributed - mitigate hotspot key problem. 