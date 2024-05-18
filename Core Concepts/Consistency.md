### Quorum consensus 
Guarantees consistency for read and write - need a coordinator
N - no of nodes
W - write acknowledgement required from W replicas 
R - read from R replicas 
If write heavy system, keep W low
If read heavy system, keep R low
If W + R > N -> strong consistency is guaranteed

### Consistency model:
1. Strong consistency - client never sees out-of-date data.
2. Weak consistency - read operations may not see the most updated value
3. Eventual consistency- given enough time, all updates are propagated
4. Sequential consistency 

### Failure Detection:
Gossip protocol - each node communicates its status to a subset of nodes. This is decentralized failure detection, better than all-to-all multicasting (which would be inefficient if there are many servers in the system)
#### Handling Temporary Failures:
Sloppy quorum - ignore offline servers in R and W. If a server is unavailable, another server will process requests temporarily. When the down server is up, changes will be pushed back to achieve data consistency. This is called "hinted handoff"
#### Handling Permanent Failures:
We implement an anti-entropy protocol to keep replicas in sync. Anti-entropy involves comparing each piece of data on replicas and updating each replica to the newest version. A Merkle tree is used for inconsistency detection and minimizing the amount of data transferred. Every non-leaf node is labeled with the hash of the values of its child nodes. You can traverse the tree to find which buckets are not synchronized and synchronize those buckets only. Using Merkle trees, the amount of data needed to be synchronized is proportional to the differences between the two replicas, and not the amount of data they contain. 

### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
