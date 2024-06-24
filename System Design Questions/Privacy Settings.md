### Problem Description
Design a system to manage privacy settings of posts and profile

### Questions to ask
1. What levels of access are allowed 

### Functional Requirements
1. Define visibility rules on profile fields 
2. Define visibility rules on posts 
3. Visibility levels - public, friends, friend-of-friend, custom groups, private 
4. Add / remove friends 
5. Search results should respect privacy settings 

### Non Functional Requirements
1. Accuracy 


### Capacity Estimation


### APIs
1. canView(userId, postId)
2. canView(userId, profileId, field)
3. profile(userId, profileId)

### Object Model


### HLD
![[Privacy Settings Design.png]]


### Database Choices


### Interesting Algorithms/Approaches


### Scaling Challenges


### Deep Dive Topics
1. Delete - invalidate entry in social graph and any feed reads should double check permissions 


### References