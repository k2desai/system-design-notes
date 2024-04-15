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
3. Online presence
4. Multiple device support - same account can be logged into multiple devices 
5. Push notifications 

### Non Functional Requirements
1. Low delivery latency 

### Capacity Estimation
1. 50 million DAU 

### APIs


### Object Model


### HLD


### Database Choices


### Interesting Algorithms/Approaches
1. [[Communication Protocols#Long Polling]] is inefficient. We use [[Communication Protocols#Websockets]] between client and server. 

### Scaling Challenges


### Deep Dive Topics


### References

System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-the-system-design-interview/B8R22v0wqJo
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-whatsapp