### Problem Description


### Questions to ask


### Functional Requirements
1. Rask scheduling
2. Priority based execution
3. Task gating for drop / pause tasks of a specific type
4. Track task status 
5. At-least once execution
6. No concurrent task execution

### Non Functional Requirements
1. Isolation - tasks ad their resources are isolated
2. Low latency - tasks start and status updates within 5 secs
3. High availability

### Capacity Estimation


### APIs


### Object Model


### HLD
![[Job scheduler design.png]]

| Component                       | Detail                                                                                                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend                        | Receives scheduling requests from clients and schedules tasks by interacting with the task store                                                                |
| Task store                      | tasks are stored in and triggered from the task store. And db with index querying capabilities                                                                  |
| Store consumer                  | Polls the db to find tasks to schedule (new or failed tasks)                                                                                                    |
| Queue                           | Acts as buffer between consumer and controller. Separate queue for each priority                                                                                |
| Controller                      | One controller process per worker host. Pulls tasks from limited set of queues. It manages priorities. Distributes among local queue for executors. Pull based. |
| Executor                        | Multiple threads responsible for task execution. Pull based.                                                                                                    |
| Heartbeat and status controller | Checks if task has been submitted, is running and is complete                                                                                                   |

### Database Choices


### Interesting Algorithms/Approaches
1. Tasks need to be idempotent (can run multiple times) and resilient (so they could die and restart at any time). Task should mark itself as successful, retriable or fatally terminated to avoid misbehavior such as infinite retries.
2. A status update has to happen every x seconds and it also updates the timestamp of the task - this way if timestamp has not been updated beyond x time interval, then it means that the task can be rescheduled because it could have been dropped somewhere. This ensures atleast once execution
3. To avoid concurrent execution, we have a heartbeat being set while process is running. If an executor is unable to send a heartbeat for a particular interval, it should kill the task because it would mean some other worker is going to pick up the task. 
4. The task poll frequency can be tuned according to desired latency. 

### Scaling Challenges


### Deep Dive Topics
1. Chaining (dependency) and periodic execution 
2. Could have maximum task retries after which it goes into dead letter queue. 

### References
https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox
https://leetcode.com/discuss/general-discussion/1082786/System-Design%3A-Designing-a-distributed-Job-Scheduler-or-Many-interesting-concepts-to-learn
https://www.youtube.com/watch?v=ta5x62cDxf4&ab_channel=SystemDesignFightClub 