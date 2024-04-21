### Problem Description


### Questions to ask
1. How many DAU
2. What features should we focus on -> direction, navigation, ETA, map rendering
3. How large is road data? Do we have access to it?
4. Should our system take traffic conditions into consideration
5. What about different travel modes like walking, driving, public transport
6. Should it support multi-stop directions
7. How about business places and photos

### Functional Requirements
1. User location update
2. Navigation service including ETA
3. Map rendering

### Non Functional Requirements
1. Accuracy - users should not be given wrong directions 
2. Smooth navigation and map rendering
3. Limit data and battery usage 
4. General availability and scalability 

### Capacity Estimation
1. 1 billion DAU - avg 5 minutes navigation/day -> 5 billion minutes/day
2. Send location every 15 sec -> 20 billion requests/day -> 200k QPS
4. Map of the world - 100PB 

### APIs
1. POST /v1/locations
2. GET /v1/nav?origin=<>&destination=<>

### Object Model


### HLD
![[Google Maps Design.png]]

| Component          | Detail                                                                                                                                                                                                                                                                                    | Technology                                                                                                                                                                                  |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Location service   | Location is used for:<br>1. Monitor live traffic - accurate ETA updates and rerouting<br>2. Detect new/closed roads<br>3. User personalization                                                                                                                                            | Cassandra for high write volume. HTTP with keep-alive option                                                                                                                                |
| Navigation service | Uses geoencoding service to convert user source/destination to lat/long and then the shortest path service uses traffic data, user preferences and filters, ETA calculations to come up with optimal path. Would need adaptive ETA and rerouting if user switches direction while driving | Websockets for bidirectional data while driving                                                                                                                                             |
| Routing tiles      | Road dataset contains associated metadata like names, latitude, longitude etc that is not organized as graphs. We have a periodic process that converts this data into routing tiles. Uses routing tiles and user preferences to come up with paths                                       | Graph data is represented as adjacency list is memory, but expensive to store that way in a database. More efficient to store in S3 organized by geohashes of lat/long pair for fast lookup |
| Geocoding service  | To store places and their corresponding lat/long pair                                                                                                                                                                                                                                     | key-value db like Redis for fast reads                                                                                                                                                      |

### Database Choices


### Interesting Algorithms/Approaches
1. Map rendering and tiling - instead of rendering the entire map as one image, the world is broken up into tiles. There are distinct set of tiles for every zoom level. The client chooses tiles as per the zoom level and only downloads relevant tiles for the area user is in and stitches them together. This provides the right level of detail without consuming excess bandwidth. Depending on user location and zoom level, fetch the appropriate images from CDN. 
	1. Could either have logic on client side to get the URL for the images on the CDN 
	2. Could have a service that client invokes to get the URLs (Map Tile Service)
2. Navigation algorithms are based on graphs and the performance of pathfinding is based on graph size. Representing the entire world as a single graph would consume too much memory - the graph needs to be broken down into manageable units for the algorithms to work. There are three sets of routing tiles for different level of detail - small and local roads, arterial roads and highways 
(Map tiles are images, routing tiles are binary files of road data)

### Scaling Challenges


### Deep Dive Topics


### References