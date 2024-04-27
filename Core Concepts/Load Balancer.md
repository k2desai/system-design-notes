### Routing algorithm:
1. Round robin - rotates requests evenly across all servers. Does not consider server load 
2. Least Connections - sends requests to the server with fewest connections 
3. IP hash - routes based on IP - same IP goes to same server for each request 
4. Load Based - Periodically queries backend servers about their load and adjusts traffic accordingly 

### Layer:
1. Layer 4 (Transport layer - TCP)
2. Layer 7 (Application layer - HTTPS) - routes based on content like url / http headers 