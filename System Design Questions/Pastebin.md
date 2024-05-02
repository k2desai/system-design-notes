### Problem Description


### Questions to ask


### Functional Requirements
1. Users should be able to upload data and get a unique url to access it
2. Users will only be able to upload text 
3. Data and links expire after a specific time, users can specify expiry
4. Users should be able to pick custom alias for their paste

### Non Functional Requirements
1. Reliable
2. Highly available
3. Low latency 
4. Not predictable links 

### Capacity Estimation
1. 5:1 read:write, 1 million pastes, 5 million reads daily. 12 pastes/sec, 58 reads/sec
2. 10 KB paste -> 10 GB/day -> 36 TB for 10 years
3. 3.6 billion pastes in 10 years, base64 encoding needs 6 chars -> 3.6 billion * 6 -> 22 GB

### APIs
1. addPaste(api_dev_key, paste_data, custom_url=None user_name=None, paste_name=None, expire_date=None)
2. getPaste(api_dev_key, api_paste_key)
3. deletePaste(api_dev_key, api_paste_key)4. 

### Object Model


### HLD
![[Pastebin Design.png]]


### Database Choices
1. Separate database for metadata (relational or dynamoDB/Cassandra) and S3 for paste contents, allows them to scale individually

### Interesting Algorithms/Approaches
Key Generation Service

### Scaling Challenges


### Deep Dive Topics
See [[URL Shortening Service]]

### References
https://www.educative.io/courses/grokking-the-system-design-interview/3jyvQ3pg6KO 