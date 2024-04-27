### Problem Description


### Questions to ask
1. What type of notifications does the system support?
2. Is it a realtime system
3. What are the supported devices 
4. What triggers notifications 
5. Will users be able to opt-out 
6. How many notifications each day?

### Functional Requirements
1. Support Push Notifications, SMS, email 
2. Support iOS, Android, desktop 

### Non Functional Requirements
1. Soft realtime - few seconds delay of notification is acceptable

### Capacity Estimation
1. 10 million mobile notifications, 1 million SMS, 5 million emails

### APIs


### Object Model


### HLD
![[Notification Design.png]]

| Component               | Detail                                                                                                                                                                                                                                                          |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DB                      | When the user signs up on our app for the first time, collect user info, notification preferences and store them to the database.                                                                                                                               |
| Notification Servers    | Provides APIs for microservices to trigger notifications, carry out basic validation, lookup user preferences and builds payloads for third party notification services. Also does some rate limiting to avoid sending too many notifications to a single user. |
| Push Notification Queue | Buffer when high volume of notifications need to be sent. Each notification type (SMS, email, push) has its own queue.                                                                                                                                          |
| Workers                 | Pull notification messages from the queue and send to appropriate third party                                                                                                                                                                                   |
| Notification log        | Notifications can be delayed or reordered but never lost, so the notifications are persisted in a database and workers implement a retry mechanism. This guarantees atleast once delivery.                                                                      |
| Notification template   | To avoid building every notification from scratch - preformatted notification that can be customized with your data. Helps to maintain consistent format and saves time.                                                                                        |
| Analytics Service       | It implements event tracking for metrics such as open rate, click rate, which is important to track engagement and understand customer behavior.                                                                                                                |


### Database Choices


### Interesting Algorithms/Approaches


### Scaling Challenges


### Deep Dive Topics
1. iOS push notifications - use Apple Push Notification Service (APNS) - a remote service provided by Apple to propagate push notifications to iOS devices. It needs a device token and a payload. 
2. Android push notifications - use Firebase Cloud Messaging (FCM)
3. SMS messages - third party services like Twilio and Nexmo 
4. Email services - Sendgrid and Mailchimp 

### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions 