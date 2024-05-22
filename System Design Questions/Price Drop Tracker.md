### Problem Description


### Questions to ask


### Functional Requirements
1. User should be able to setup alerts for price changes 
2. User should get notified in case of any price drop below threshold
3. Support multiple websites 
4. Show history of product

### Non Functional Requirements
1. Alert within a few hours of price change 
2. High availability 
3. Reliability - should not lose alerts / data 

### Capacity Estimation
1. 10 million users -> 5 million alerts setup / day -> 50 QPS -> peak 200 QPS
2. 5 million unique products -> queries 4 time/day -> 20 million queries/day -> 200 QPS

### APIs
1. POST /setup_alert
```{
	url : "",
	threshold_price : ""
}
```

### Object Model
1. User
2. Product

### HLD
![[Price Drop Tracker design.png]]

### Database Choices


### Interesting Algorithms/Approaches
1. Stale price scanner could have some adaptive interval rate for scanning fast changing prices more often than the slow ones. Also - it can change the status between scanning and scanned when it is going through a product so that the same product does not get scanned simultaneously. 
2. Notifiers create a separate message for each user to be notified and put onto queue. 

### Scaling Challenges


### Deep Dive Topics


### References