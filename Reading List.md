
### System Design

- [ ] Database Internals
- [ ] Alex Xu System Design
- [ ] Alex Xu System Design Vol II
- [ ] Designing Data Intensive Applications

### Leadership

- [ ] Staff Engineer

### Interview Format
1. Start your interview by defining the functional and non-functional requirements. For user facing applications like this one, functional requirements are the "Users should be able to..." statements whereas non-functional defines the system qualities via "The system should..." statements.    
2. Prioritize the top 3 functional requirements. Everything else shows your product thinking, but clearly note it as "below the line" so the interviewer knows you wont be including them in your design. Check in to see if your interviewer wants to move anything above the line or move anything down. Choosing just the top 3 is important to ensuring you stay focused and can execute in the limited time window.    
3. When evaluating non-functional requirements, it's crucial to uncover the distinct characteristics that make this system challenging or unique. To help you identify these, consider the following questions:    
    1. **CAP theorem:** Does this system prioritize availability or consistency? Note, that in some cases, the answer is different depending on the part of the system         
    2. **Read vs write ratio:** is this a read heavy system or write heavy? Are either notably heavy for any reason?        
    3. **Query access pattern:** Is the access pattern of the system regular or are their patterns or bursts that require particular attention. For example, the holidays for shopping sites or popular events for ticket booking.
4. Do not directly jump into scaling. State you would 1) **Benchmark/Load Test**, 2) **Profile** for bottlenecks 3) address bottlenecks while evaluating alternatives and trade-offs, and 4) repeat. See [Design a system that scales to millions of users on AWS](https://github.com/donnemartin/system-design-primer/blob/master/solutions/system_design/scaling_aws/README.md) as a sample on how to iteratively scale the initial design.

### Talking Points:

Scaling the database: 
• Vertical scaling vs Horizontal scaling 
• SQL vs NoSQL 
• Master-slave replication 
• Read replicas 
• Consistency models 
• Database sharding 

Other talking points: 
• Keep web tier stateless 
• Cache data as much as you can 
• Support multiple data centers 
• Lose couple components with message queues 
• Monitor key metrics. For instance, QPS during peak hours and latency
• Load balancer - round robin / least connections

Non-functional requirements
- Scalability
- Consistency
- Availability 
- Reliability
- Performance 
- Latency
- Read vs write heavy

API Gateway:
1. Request validation
2. Authentication / authorization
3. TLS SSL termination
4. Server side encryption
5. Caching
6. Rate limiting
7. Request dispatching
8. Request deduplication
9. Usage data collection

Availability:
1. If S3 - deploy across different availability zones 
2. DNS will take care of routing to other region if one region is down 

### Open Questions:
1. What is QPS / RPS of a typical server? 64000? - https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/requirements-of-tinyurls-design