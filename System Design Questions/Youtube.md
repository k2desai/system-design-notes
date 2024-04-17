### Problem Description
> Design a system to store and serve videos. Use cases to cover:
> - Upload videos
> - Search for videos
> - View videos
> - Analytics on videos
> - Comment/Like/Dislike Video
> 
### Questions to ask
1. What features are important
2. What clients do we need to support?
3. How many DAU
4. What is the average daily time spent on the product 
5. Do we need to support international users
6. What are the supported video resolutions
7. Is encryption required?
8. Any file size requirement for videos?
### Functional Requirements
1. Ability to upload videos 
2. Smooth video streaming
3. Ability to change video quality
4. Ability to search for videos 
5. Like / dislike / comments

### Non Functional Requirements
1. Low infrastructure cost
2. High availability / scalability / reliability
3. Eventual consistency 

### Capacity Estimation
1. 5 million DAU 
2. Users watch 5 videos / day 
3. 10% users upload 1 video / day
4. Video size - 300 MB - space = 5 million * 10% * 300 MB = 150 TB

### APIs


### Object Model


### HLD

![[Pasted image 20240417120417.png]]


| Component      | Details                                                                                                               |
| -------------- | --------------------------------------------------------------------------------------------------------------------- |
| Upload storage | Temporarily holds videos before preprocessing                                                                         |
| Bigtable       | To store thumbnails because of high throughput and scalability for key-value data. Optimal for data items below 10 MB |
| Encoders       | For video postprocessing, uploading to CDN and colocation servers.                                                    |
| CDNs           | CDNs can hold the more popular videos for their respective regions                                                    |


### Database Choices
1. Video and user metadata - Relational db - with primary-secondary replication. 

### Interesting Algorithms/Approaches
1. Chunked encoding: Since multiple devices could be used to stream the same video, we may have to encode the same video using different encoding schemes resulting in one raw video file being converted into multiple files each encoded differently. We divide the video into segments and process each segment to generate chunks. The choice of encoding scheme for a segment will be based on the detail within the segment to get **optimized quality with lesser storage requirements**.
2. Adaptive streaming: While the content is being served, the bandwidth of the user is also being monitored at all times. Since the video is divided into chunks of different qualities, each of the same time frame, the chunks are provided to clients based on changing network conditions. When the bandwidth is high, a higher quality chunk is sent to the client and vice versa
3. Video deduplication needed? Chunking would help with that if parts of video are duplicates

### Scaling Challenges


### Deep Dive Topics


### References

1. https://bytebytego.com/courses/system-design-interview/design-youtube
2. https://medium.com/@sureshpodeti/system-design-youtube-f5077b6de0f1
3. https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-youtube
4. https://www.educative.io/courses/grokking-the-system-design-interview/xV26VjZ7yMl
