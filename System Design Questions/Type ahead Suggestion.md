### Problem Description


### Questions to ask
1. Is matching supported at beginning of query or middle as well?
2. How many autocomplete suggestions must the system return?
3. How does the system know which 5 to return?
4. Does the system support spell check?
5. Are search queries in English?
6. Do we allow capitalization and special characters?
7. How many users use the product?

### Functional Requirements
1. As user types in a query, we return top 5 terms starting with whatever the user has typed. 
2. Relevant autocomplete suggestions sorted by popularity 

### Non Functional Requirements
1. Fast response time - within 100 ms
2. Scalable
3. Highly available 
4. Fault tolerant 

### Capacity Estimation
1. 10 million DAU, 10 searches/day, 20 bytes per query string, 20 requests for each search query -> 24000 QPS, peak QPS 48000
2. 20% daily queries are new -> 0.4 GB daily

### APIs
1. getSuggestions(prefix)
2. addToDatabase(query)

### Object Model


### HLD
![[Typeahead Design 1.png]]
![[Typeahead Design 2.png]]

| Component              | Detail                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Data gathering service | Gathers user input queries and aggregates them. Analytics logs hold raw search data, which is then aggregated and used by Workers to build the Trie. You could perform the Trie update weekly                                                                                                                                                                                                                                                                                                                |
| Query service          | Given a search query prefix, return 5 most frequently searched terms                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Filter                 | We have to remove hateful, violent, sexually explicit, or dangerous autocomplete suggestions. We can add a filter layer in front of the Trie Cache to filter out unwanted suggestions. It also gives us the flexibility of removing results based on different filter rules - freshness, user location, language, demographics, personal history. Unwanted suggestions are removed physically from the database asynchronically so the correct data set will be used to build trie in the next update cycle. |
| Trie Cache             | Caching the top searched terms will be extremely helpful as there will be a small percentage of queries that will be responsible for most of the traffic.                                                                                                                                                                                                                                                                                                                                                    |

### Database Choices
1. Trie could be stored in MongoDb or a key-value store where every prefix is a key, and the top 5 suggestions are the values. 

### Interesting Algorithms/Approaches
1. Trie with top 5 most frequent queries stored at each node. 
2. If top searches are different in different countries, we might build different tries for different countries. To improve the response time, we can store tries in CDNs.
3. We can have Map-Reduce running periodically to create updated Trie. We can make a copy of the Trie on each server and switch once new Trie is ready. Or we could have a primary-secondary configuration and update the secondary while primary is serving traffic, once update is complete, we can make the secondary our new primary. 
4. We need to add new frequencies and subtract frequencies for time periods no longer included. We can add and subtract frequencies based on Exponential Weighted Moving Average of each term. In EMA, we give more weight to the latest data.

### Scaling Challenges
1. Client Side:
	1. Only hit server if user has not pressed any key for 50 ms. Wait initially until user has entered a few characters. 
	2. Since query service needs a very fast speed, for web applications, browsers usually send AJAX requests to fetch autocomplete results. The main benefit of AJAX is that sending/receiving a request/response does not refresh the whole web page.  It allows web pages to exchange a small amount of data with the server without interfering with the display and behaviors of the existing web page.
	3. Autocomplete search suggestions may not change much within a short time, can be saved in browser cache
	4. Establish a connection with the server as soon as the user visits the search page. so no waste of time establishing the connection when the user inputs the first character. Usually, the connection is established with the server via websockets.
2. Data sampling is also important - could consider that only 1 out of N requests is logged by the system. If we don’t want to show a term which is searched for less than 1000 times, it’s safe to log every 1000th searched term.
3. When the trie grows too large to fit in a single server, you can shard based on starting characters. To mitigate data imbalance, look at historical patterns and apply a smarter logic. 

### Deep Dive Topics


### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-the-system-design-interview/mE2XkgGRnmp
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-the-typeahead-suggestion-system
https://medium.com/@prefixyteam/how-we-built-prefixy-a-scalable-prefix-search-service-for-powering-autocomplete-c20f98e2eff1 