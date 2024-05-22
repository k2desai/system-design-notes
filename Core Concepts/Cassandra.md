In Cassandra, there is no masters, all nodes have same data with eventual consistency. You can use cassandra for write-heavy workflow like feeds, activity, etc. It uses LSM tree (which has append only log) so is not very read friendly (as opposed to dynamoDB which uses BTree and is better for reads).

But in cassandra you can tune consistency levels, whereas dynamodb is always eventual consistency with sloppy quorum

GraphDb has scalability issues (partitioning is a premium feature, special SQL syntax is the primary benefit )