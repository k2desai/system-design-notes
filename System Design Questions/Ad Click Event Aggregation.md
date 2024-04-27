### Problem Description
> Problem Description

### Questions to ask


### Functional Requirements
1. Users can click on an ad and be redirected to a website
2. Advertisers can query ad metrics - number of clicks in last x minutes and top N most clicked ads in last x minutes

### Non Functional Requirements
1. Scalable to support 10000 clicks/sec
2. Low latency analytics queries 
3. Fault tolerant and accurate data collection 
4. As realtime as possible
5. Idempotent click tracking 

### Capacity Estimation
1. 10 million ads
2. 100 million clicks/day
3. Peak 10000 clicks/sec

### APIs
1. POST /click
```{
"ad_id" : "123"
}
```
2. GET /metrics?ad_ids={ad_id[]}&start_time={start_time}&end_time={end_time}&interval={interval}

### Object Model
1. Ad Click Event
2. Advertisement
3. Aggregated Click Data

### HLD
![[Ad Click Event Aggregator Design.png]]

| Component       | Details                                                                                                                                                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Click Processor | Rather than put the URL in the ad for the user to navigate to (and missing click data), a click on the ad will send a request to Click Processor. This will track the client and respond with 302 redirect. Need to make sure it can handle the load                   |
| Flink           | Aggregates the streaming data as per specified interval and writes to OLAP DB - more flexible alternative than Spark (where the job needs to be configured to run at an interval), also Flink can flush every few seconds, even if it aggregates in 1 minute interval. |
| Reconciliation  | Losing click data means we lose revenue, so in addition to streaming, we also write raw data to S3/Cassandra and run a Spark process to recalculate and reconcile the metrics.                                                                                         |

### Database Choices


### Interesting Algorithms/Approaches
1. To ensure we dont get duplicate data, the Ad Placement service will generate a unique impression id that can be used as idempotency key. We need to make sure we dedupe BEFORE we put the click in the stream (else if duplicate comes in different aggregation windows, it could get counted twice). We can use a distributed cache like Redis for caching the keys received. 
2. We could run cron jobs to aggregate data at different granularities, to ensure user is able to load it fast. We are trading storage space for query performance. 
3. We could also extend the aggregation window by a few seconds to try to capture some late-arriving events. There is a trade-off between latency and accuracy here. 
4. This is lambda architecture where we have both batch and streaming paths. In Kappa architecture we combine batch and streaming into one path. 
5. Using event time (passed from client side, but can be a problem if client machine is set to a different time) is more correct than processing time (since server could get the click event much later)

### Scaling Challenges
1. Stream and stream processors can be partitioned by ad id. OLAP database can be partitioned by advertiser id. If an ad is determined popular by ad spend/click volume, then further partition it by appending a random number to ad id. 

### Deep Dive Topics


### References