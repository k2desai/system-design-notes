### Problem Description
A web crawler is used for many purposes:
1. Search engine indexing 
2. Web archiving - used by national libraries
3. Web mining - to discover useful content from the internet - eg. financial firms crawl shareholder meetings and annual reports 
4. Web monitoring - to monitor copyright infringement 

### Questions to ask
1. What is the purpose of the crawler?
2. How many pages to collect per month?
3. What is the content type included -> HTML 
6. How do we handle web pages with duplicate content 

### Functional Requirements
1. Search engine indexing 
2. Consider newly added / edited pages 
3. Store the pages crawled, ignoring dupes 

### Non Functional Requirements
1. Scalable - should be extremely efficient using parallelization 
2. Robustness - Handle bad html, unresponsive servers, malicious links, crashes, etc 
3. Politeness - should not make too many requests to a website 
4. Extensibility - support new content type with minimal changes 

### Capacity Estimation
1. 1 billion web pages / month -> 400 pages/sec. Peak QPS - 800
2. Average webpage size 500 KB -> 500 TB / month -> 30 PB for 5 years  
3. 100 ms / page -> 100 million seconds -> 1000 days -> 3 years to traverse all pages 

### APIs


### Object Model


### HLD
![[Web Crawler Design.png]]

| Component       | Detail                                                                                                                                                                                                                                                                                                                                                   |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Seed URLs       | Starting point for the crawl process - select based on locality or topics to traverse as many pages as possible                                                                                                                                                                                                                                          |
| URL Frontier    | FIFO queue of URLs to be downloaded. Ensures <br>1. Prioritization - higher ranking pages are put on queues from which URLs are picked with higher probability<br>2. Politeness - by having separate queues for every page and downloading one page at a time, with some delay, from a given page<br>3. Freshness - Recrawl based on page update history |
| HTML Downloader | Downloads the webpages (provided by URL Frontier) from the internet. Uses DNS resolver to translate the url into IP address. It looks at Robots Exclusion Protocol - robots.txt - can cache this file - which specifies what pages are allowed to download                                                                                               |
| Content Parser  | Parse and validate the downloaded web page - malformed web pages could cause problems                                                                                                                                                                                                                                                                    |
| Content Seen    | 30% web pages have duplicated contents - eliminate data redundancy - compare checksum hash values of two web pages                                                                                                                                                                                                                                       |
| Content Storage | Most of the content is stored on disk, popular content is in memory to reduce latency. This would be a blob store.                                                                                                                                                                                                                                       |
| Link extractor  | Parses and extracts links from HTML pages - relative paths are converted to absolute urls                                                                                                                                                                                                                                                                |
| URL Filter      | Excludes certain content type, file extensions, error links and blacklisted URLs                                                                                                                                                                                                                                                                         |
| URL Seen        | Keeps track of URLs visited before or already in the Frontier - avoids adding same URL multiple times. If URL is already in the storage, it does nothing, else it adds it to the frontier.                                                                                                                                                               |
| URL Storage     | Stores already visited URLs. Can be a relational db, also stores last time it was visited. The scheduler can pick URLs from here depending on time and update frequency.                                                                                                                                                                                 |

### Database Choices


### Interesting Algorithms/Approaches
1. DFS vs BFS - Depth of DFS can be very deep. BFS is more commonly used. However many links on a page are internal links - the server of that website would be flooded with requests ("impolite"). It also does not take any priority into consideration (as per update frequency, page rank - to measure usefulness, web traffic, etc). 
2. Spider traps can cause the crawler to go into infinite loops. For instance, an infinite deep directory structure is listed as follows: www.spidertrapexample.com/foo/bar/foo/bar/foo/bar/…. Can be avoided by setting maximum length of URL. 
3. Server-side rendering: Numerous websites use scripts like JavaScript, AJAX, etc to generate links on the fly. If we download and parse web pages directly, we will not be able to retrieve dynamically generated links. To solve this problem, we perform server-side rendering (also called dynamic rendering) first before parsing a page
4. Filter out unwanted pages: With finite storage capacity and crawl resources, an anti-spam component is beneficial in filtering out low quality and spam pages
5. Checkpointing: A crawl of the entire Web takes weeks to complete. To guard against failures, our crawler can write regular snapshots of its state to the disk. An interrupted or aborted crawl can easily be restarted from the latest checkpoint.

### Scaling Challenges
1. Crawl jobs are distributed among multiple servers. The Frontier could distribute URLs to multiple queues. Distribute crawl servers geographically to make them closer to website hosts. 
2. DNS resolver results can be cached. 
3. To avoid long wait time, a maximal wait time is specified. If a host does not respond within a predefined time, the crawler will stop the job and crawl some other pages.

### Deep Dive Topics


### Drilldown
1. Multiple queues in frontier for handling priority and then politeness. Freshness. 
2. Client side browser caching and AJAX requests
3. Spider traps


### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.educative.io/courses/grokking-the-system-design-interview/NE5LpPrWrKv
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-web-crawler
