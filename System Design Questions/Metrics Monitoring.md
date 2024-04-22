### Problem Description


### Questions to ask
1. Are we building in-house system or SaaS service like Datadog, Splunk, etc 
2. What metrics do we want to collect?
3. What is the scale of the infrastructure we are monitoring with this system?
4. How long should we keep the data?
5. May we reduce the resolution of the metrics for long term storage
6. What are the supported alert channels
7. Do we need to collect logs such as error / access logs
8. Do we need to support distributed systems tracing?

### Functional Requirements
1. CPU usage
2. Request count
3. Memory usage
4. Message count in message queues
5. Log monitoring - Out of scope
6. Distributed system tracing - Out of scope (we could do this by having a distributed counter and appending that auto-incrementing unique id to every message for that request)

### Non Functional Requirements
1. Scalability - accommodate growing metrics and alert volume
2. Low latency - for dashboard and alerts
3. Reliability - avoid missing critical alerts
4. Flexibility - to integrate new technologies in the future

### Capacity Estimation
1. 100 million DAU
2. 1000 server pools, 100 machines/pool, 100 metrics/machine -> 10 million metrics
3. 1 year data retention -> raw for 7 days, 1 minute for 30 days, 1 hour for 1 year

### APIs


### Object Model
Metrics data is recorded as a time series data with a set of values and associated timestamps. The series can be identified by its name and a set of labels - Eg. 
	metric_name = cpu_load
	label = host:i631,env:prod
	timestamp = 1613707265
	value = 0.29

### HLD
![[Metrics Monitoring Design.png]]

![[Alerting Design.png]]

| Component         | Detail                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Metrics collector | Data loss is not the end of the world. Can have push or pull model. With pull model (Eg. Prometheus), it can use service discovery like Zookeeper (which can also have rules about when to collect metrics) and the hit an http endpoint to collect metrics. With a push model, a collection agent is installed on every server being monitored and it is responsible for pushing metrics to the collector. |
| Query service     | Makes it easy to query and retrieve the data from the timeseries database. It is a very thin wrapper if we choose a good timeseries database and can also be replaced with the database's own query interface. But having a service decouples the database from the clients and gives flexibility to change either.                                                                                         |
| Kafka             | To reduce dataloss if timeseries database is unavailable, we have a queuing component to process and push data to the database. It helps decouple data collection and processing services.                                                                                                                                                                                                                  |
| Alerting system   | Alerting rules are configured in yaml files and loaded to cache. The alert manager loads metrics at a predefined interval and stores them in Cassandra as the alert store. Also publishes to Kafka for different consumers to consume. Alerting - PagerDuty, Visualization - Grafana                                                                                                                        |


### Database Choices
Write load is heavy, while read load is spiky. Relational database is not optimized for operations on timeseries data and heavy write loads. Non-relational databases could handle the write load, but may not work well with timeseries data. There are storage systems optimized for timeseries data with custom query interfaces for timeseries analysis and features for data retention and aggregation. OpenTSDB (based on Hadoop), Twitter uses MetricsDB, Amazon offers Timestream. InfluxDB and Prometheus are the most popular ones - they rely on in-memory cache and on-disk storage. 

### Interesting Algorithms/Approaches
1. Pull vs Push - 
	1. Pull:
		1. Easy debugging - can pull metrics any time
		2. If application does not respond to request for metrics, can quickly check if it is down
	2. Push:
		1. Missing metrics could be because of network issues
		2. Good for short lived batch jobs since they may not live long enough to be pulled
		3. Need to whitelist which servers you can accept metrics from 
2. Metrics aggregation can happen in 
	1. Collection agent on client side
	2. Ingestion pipeline before writing to storage - Write volume will reduce, but handling late events can be a problem and we lose data precision and flexibility
	3. Query side after writing to db - No data loss but query speed might be slow as it aggregates at runtime
3. Space optimization:
	1. Store only delta in the time series
	2. Downsampling historical data
	3. Cold storage for inactive data

### Scaling Challenges
1. In a pull model, we can scale our metrics collectors by using consistent hashing to ensure that one server is mapped to only one collector. We could use the server's unique name to identify its place in the hash ring. With a push model, we could put a load balancer in front of the collector cluster and auto scale based on load. 
2. Can partition Kafka queue by metric name and then by label 

### Deep Dive Topics


### References
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-distributed-logging
System Design Interview An Insiders Guide Volume 2