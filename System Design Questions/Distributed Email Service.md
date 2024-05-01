### Problem Description
> 

### Questions to ask
1. How many users 
2. How do users connect with mail servers?
3. Can mails have attachments

### Functional Requirements
1. Send and receive emails
2. Fetch all emails
3. Filter emails by read and unread status 
4. Search emails by sender / subject / body
5. Anti-spam and anti-virus alert

### Non Functional Requirements
1. Reliability 
2. Availability
3. Scalability
4. Flexibility and extensibility

### Capacity Estimation
1. 1 billion users, send 10 emails per day -> 100,000 QPS
2. Receive 40 emails/day, email - 50 KB, 1 year -> 1 billion * 40 * 50 KB * 365 -> 730 PB
3. If 20% emails have 500 KB attachment -> 1460 PB attachment

### APIs
1. POST /v1/messages
2. GET /v1/folders
3. GET /v1/folders/{folder_id}/messages - needs to support [[Pagination]]4. 
4. GET /v1/messages/{message_id}

### Object Model


### HLD
![[Email Design.png]]

| Component         | Detail                                                                                                                                                                                                                                                                                                                                                                                                            |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Webmail           | Used by web browsers to send/receive mails                                                                                                                                                                                                                                                                                                                                                                        |
| Web servers       | Public facing servers that manage login, signup, sending mail, loading mails. Responsible for email validation. If domain of receiver is different, then put the message on outgoing queue for SMTP workers to pick and send to recipient mail server. Decouples SMTP workers so they can scale independently. Users communicate with web server closest to them, data is replicated across multiple data centers |
| Realtime servers  | Pushing new email updates to online clients in realtime - websockets with fallback to long polling if web sockets are not supported                                                                                                                                                                                                                                                                               |
| Metadata DB       | Stores subject, body, users                                                                                                                                                                                                                                                                                                                                                                                       |
| Distributed cache | Cache recent emails repeatedly loaded by client                                                                                                                                                                                                                                                                                                                                                                   |
| Search store      | Inverted index to support fast search                                                                                                                                                                                                                                                                                                                                                                             |

![[Email Sending Design.png]]
![[Email Receiving Design.png]]
### Database Choices
1. Characteristics of email metadata:
	1. Headers are small and frequently accessed, body is bigger but infrequently accessed. 
	2. Fetching, searching, marking read is isolated to a single user
	3. Data recency impacts data usage
	4. High reliability requirement
2. Relational is not good for email body, S3 is good for backup but not good for marking read, searching, etc. Cassandra may be a good option 
3. Gmail (Bigtable - not open source) / Outlook have their own custom databases
4. Partition by user id, with folder id as a clustering key within the partition. Every user has a single primary because consistency is important 

### Interesting Algorithms/Approaches
1. Email protocols:
	1. SMTP (Simple Mail Transfer Protocol) - sending emails from one mail server to another
	2. POP - receive and download emails from remote mail server to local email client 
	3. IMAP - receiving email for local email client, only download when you click on it
	4. HTTPS - used to access mailbox for web based email
	5. DNS - used to lookup mail exchanger (MX) record for recipient's domain
	6. MIME - multipurpose internet mail extension - allow attachments to be sent over internet
2. Email search is different from web search, since it only has to search users mailbox, results need to be sorted by time, attachment, etc and indexing needs to be near realtime. Can either use ElasticSearch or our own custom search solution. For any search, disk I/O will be the bottleneck - so we can use Log-Structure Merge Tree to structure index data on disk

### Scaling Challenges


### Deep Dive Topics


### References
System Design Interview An Insiders Guide Volume 2