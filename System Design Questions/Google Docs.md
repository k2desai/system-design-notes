### Problem Description


### Questions to ask


### Functional Requirements
1. Document collaboration 
2. Conflict resolution
3. Suggestions 
4. View Count
5. History 

### Non Functional Requirements
1. Low latency
2. Reliability 
3. Strong Consistency for conflict resolution. 
4. Availability 
5. Scalability - large number of users should be able to use the service at the same time

### Capacity Estimation
1. 80 million DAU
2. 20 users editing document concurrently
3. Text document - 100 KB
4. 30% contain images (800 KB), 2% contain videos (3 MB)
5. A user creates one document in a day
6. 32 TB/day
7. 80 million / 64k -> 1300 servers

### APIs


### Object Model


### HLD
![[Google Docs Design.png]]

| Component      | Detail                                                                                                                                  |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Pub-sub system | Notifications, emails, view counts, and comments are asynchronous operations that can be queued through a pub-sub component like Kafka. |
|                |                                                                                                                                         |

### Database Choices
1. Relational database for saving users’ information and document-related information for imposing privilege restrictions. 
2. NoSQL for storing user comments for quicker access. 
3. Time series database to save the edit history of documents and to recover different versions of the document. 
4. Blob storage to store videos and images within a document. 
5. Distributed cache like Redis and a CDN to provide end users good performance. We use Redis specifically to store different data structures, including user sessions, features for the typeahead service, and frequently accessed documents. The CDN stores frequently accessed documents and heavy objects, like images and videos.

### Interesting Algorithms/Approaches
We need our edit operations to be:
- **Commutativity**: The order of applied operations shouldn’t affect the end result.
- **Idempotency**: Similar operations that have been repeated should apply only once.
1. Operational Transform - lock-free, non-blocking approach for conflict resolution. It uses the positional index method to resolve conflict. Disadvantages:
	1. Each operation to characters may require changes to the positional index. This means that operations are order dependent on each other.
	2. It’s challenging to develop and implement.
2. Conflict-free Replicated Data Type - complex data structure but simplified algorithm. It assigns a globally unique identity to each character and it globally orders each character. It uses fractional positional indices for this purpose. It ensures strong consistency between users. Even if some users are offline, the local replicas at end users will converge when they come back online.

### Scaling Challenges
1. Users maintain a replica of documents at their end while data is being propagated through WebSockets to end-servers. Therefore, user-perceived latency will be low. Apart from this, users mostly tend to write textual data in a document that’s small. Therefore, dissemination of data will be carried out with low latency.


### Deep Dive Topics


### References
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-google-docs