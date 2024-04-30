### Problem Description


### Questions to ask


### Functional Requirements
1. User should be able to view a list of problems
2. User should be able to view a problem and code a solution in different languages
3. User should be able to submit a solution and get feedback

### Non Functional Requirements
1. Prioritize availability over consistency
2. Support isolation and security when running user code
3. Return submission results within 5 secs 
4. Scale to support competitions with 100,000 users

### Capacity Estimation
1. 4000 problems
2. 100,000 users

### APIs
1. GET /problems?page=1&limit=100 -> Partial(Problem)
2. GET /problems/:id?language={language} -> Problem
3. POST /problems/:id/submit -> Submission { code: string, language: string }

### Object Model
1. Problem - statement, test cases
2. Submission
3. User

### HLD
![[Leetcode Design.png]]

| Component  | Detail                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API Server | Handle incoming requests from the client and return the appropriate data                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| AWS SQS    | The API server will add the submission to the queue and the containers will pull submissions off the queue as they become available. This will help us manage the load on the containers and ensure that we don't lose any submissions during peak times. This makes the system asynchronous and the user will need to poll the server for results. It is also useful in the event of a container crash or other issue that prevents the code from running successfully. We'd simply requeue the submission and try again. |

### Database Choices
1. While either a SQL or NoSQL database could work here, we choose a NoSQL DB like DynamoDB because we don't need complex queries and we can nest the test cases as a subdocument in the problem entity.

### Interesting Algorithms/Approaches
1. We run the user submitted code in a container - we can reuse the same container for multiple submissions reducing resource usage and improving performance. (Other options - use VM / Serverless functions). We need to make sure we limit network access, set CPU and memory bounds, and enforce a timeout on the code execution
2. To actually run the test cases, we need to define the serialization strategy for each data structure and ensure that the test harness for each language can deserialize the input and compare the output to the expected output. 

### Scaling Challenges
For smaller systems like this one, a monolithic architecture might be more appropriate. This is because the system is small enough that it can be easily managed as a single codebase and the overhead of managing multiple services isn't worth it. With that said, we go with a simple client-server architecture for this system. We added queues to scale during competitions - but if we ensure users register before a competition, then we can scale up our containers in advance and we dont need a queue. 

### Deep Dive Topics


### References
https://www.hellointerview.com/learn/system-design/answer-keys/leetcode 