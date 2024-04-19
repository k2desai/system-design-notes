
### Two dimensional search
If we store both, latitude and longitude, you can only index by one. So the search speed can only be improved in one direction but not the other. The high level idea of most search algorithms is to divide the map into smaller areas and build indices for faster search 

### Evenly divided grids
Problem is uneven data distribution - we want granular grids for dense areas and large grids in sparse areas

### Geohashing
Reduce two dimensional latitude and longitude into one dimensional string of letters and digits. The longer the shared prefix, the closer two items are. But the reverse is not true. There are algorithms available to calculate the neighbors. Good for range and radius queries.
![[Geohashing.png]]
![[Geohashing range.png]]

### Quadtrees
This is an in-memory tree structure built at startup time. Recursively divide a two-dimensional space into four quadrants until a certain criteria is met. Total memory requirement is of the order of 1-2 GB, generally takes a few minutes to build a quadtree for 100 million businesses. After a quadtree is built, start searching from the root node till you find the leaf node with the search point. If that node has required number of businesses, return it. Else look at the neighbors. Good to find k-closest points 

**Operational considerations:**
Due to the long server startup time, we should roll out new releases incrementally to a small subset of servers at a time. A lot of servers querying database together during startup can also overload the database. Updating quadtree on the fly requires locking and can complicate the design. 

### Google S2 


### Contraction hierarchy
Contraction hierarchy is a speed-up method optimized to exploit the properties of graphs representing road networks. The speed-up is achieved by creating shortcuts in a preprocessing phase, which are then used during a shortest-path query to skip over unimportant vertices. Source: Wikipedia

### References
https://tarunjain07.medium.com/geospatial-geohash-notes-15cbc50b329d 