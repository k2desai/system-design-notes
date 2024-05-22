### Problem Description


### Questions to ask
1. Is it 1-1 or group chat based? -> both
2. Is it a mobile app or web app -> both
3. What is the scale of the app - startup or massive scale?
4. What is the group member limit for group chats?
5. What features are important?
6. Can it support attachments?
7. Is there a message size limit?
8. Is end-to-end encryption needed?
9. How long do we need to store chat history?

### Functional Requirements
1. 1-1 chat
2. Group chat with max 100 people 
3. Send media / files / etc
4. Online presence
5. Multiple device support - same account can be logged into multiple devices 
6. Push notifications 
7. Message status - Delivered / Read / etc

### Non Functional Requirements
1. Low delivery latency 
2. Consistency and ordering of messages
3. Availability
4. Scalability

### Capacity Estimation
1. 50 million DAU 

### APIs


### Object Model


### HLD

![[Messenger Design.png]]

![[Whatsapp Design.png]]

**Chat Server Selection:**
Option 1 - Consistent Hashing
Decide a server based on a hash of user id. If any server goes down, all connections would move to the next server on the ring - might cause thundering herd - to solve this, use virtual servers on the ring, so all traffic does not get routed to a single server 

Option 2 - Load balancing using Round Robin / Load based
Regular load balancing algorithms - with the userId to server mapping being held in a database. 

Option 3 - second diagram: Client gets assigned a chat server randomly from load balancer. It registers the user on chat server registry. This registry also serves as online/offline status manager. Then when client sends a message it goes to the queue. Flink reads from that queue and checks user status - if online - it sends message to that server. If offline it sends to notification system. Flink could also batch together all messages for a particular chat server host. 

**Chatting flow:**
1. Client logs in and looks up chat server 
2. Establishes websocket connection with the selected chat server 
3. User sends a message to the chat server 
4. Message gets written to pub-sub channel (Redis pubsub / Kafka) - this pub-sub channel can be partitioned by user id 
5. In addition, it also checks if recipient is offline, if it is, then send to notification service
6. Chat server subscribes to the pub-sub for its user ids 
7. Message is received from pub-sub channel and delivered to client 
8. If the client wants to send media, it first sends the media to the media service. (Another option is uploading media directly from client side using presigned urls but for file sizes upto few MB, this might be an overkill). Media service gives back the url of the file stored in S3. Client will then send this link as part of its message. See [[Instagram#Media Service]]

**Online status:**
This can be maintained by the chat service. The status is maintained depending on if the client is connected to the chat server via websocket. While the client is connected, it will send a heartbeat and it is considered online. If the client goes offline, i.e. no heartbeats for over 10 seconds, client can be marked offline. Status is stored / updated in a distributed Redis cluster. Lookup for a user status can happen by looking up the entry in the Redis database. Setting and reading status has to be via websockets since we need to continuously set / read status as it can change anytime. 

**Saving chat history:**
When one user sends a message, user can first lookup the user / group service to get the recipient ids and include them as part of the message body. The chat service then puts this message onto the pub-sub queue. We can use Redis pub-sub with sorted sets - the key is user id and the value is the set of messages sorted by sent time. If user is offline, it will be held in the pub-sub till the user comes online. (What if sender is on the same chat server as the receiver?) When the user is online, the chat service will pull the user's messages from the pub-sub. For long-term saving of messages - it is a choice - WhatsApp does not save any messages - they are only backed up to Google Drive and retrieved from there if a user logs in from a different device. In other cases, you can choose to store the messages - what database would you use?

**Notifications:**
See [[Notification System]]


### Database Choices


### Interesting Algorithms/Approaches
1. [[Communication Protocols#Long Polling]] is inefficient. We use [[Communication Protocols#Websockets]] between client and server. 

### Scaling Challenges


### Deep Dive Topics
1. [[Notification System]]
2. [[Amazon S3#Presigned URLs]]

### References

System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-the-system-design-interview/B8R22v0wqJo
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-whatsapp
https://systemdesign.one/slack-architecture/ 
https://discord.com/blog/how-discord-stores-trillions-of-messages