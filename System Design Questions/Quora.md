### Problem Description
Design Quora, Reddit, etc 

### Questions to ask


### Functional Requirements
1. User can ask questions and give answers.
2. Questions and answers can include images and videos 
3. It is possible for users to upvote / downvote / comment on answers 
4. Users should have a search feature 
5. Users can view a feed when they login 
6. Ranking answers - most helpful should be ranked first 

### Non Functional Requirements
1. System should scale well 
2. Eventual consistency 
3. High availability and performance 

### Capacity Estimation
1. 1 billion users, 300 million DAU
2. Servers needed -> 300 million QPS / 64000 -> 4700 servers
3. Average 1 question per user per day, 100 KB / post -> 300 million * 100 KB -> 30 TB 

### APIs
1. POST - postQuestion(user_id, question, description, topic_label, video, image)
2. POST - postAnswer(user_id, question_id, answer_text, video, image) 
3. upvote(user_id, question_id, answer_id)
4. comment(user_id, answer_id, comment_text)
5. search(user_id, search_text)

### Object Model
![[Quora Design.png]]

### HLD


### Database Choices


### Interesting Algorithms/Approaches
1. Answers need a ranking system - ranking based on date is convenient, but you might want to show most relevant answers first. Relevant can be determined by a combination of upvotes and an ML algorithm to determine the most useful answer. 

### Scaling Challenges


### Deep Dive Topics


### References
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-quora 