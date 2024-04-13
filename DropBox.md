### Problem Description
Dropbox is a cloud-based file storage service that allows users to store and share files. It provides a secure and reliable way to store and access files from anywhere, on any device.

### Questions to ask
1. What are the important features -> upload, download, file sync, notifications
2. Mobile app / web app -> both
3. Supported file formats -> any
4. Do files need to be encrypted
5. Is there a file size limit
6. How many users does the product have

### Functional Requirements
1. Users should be able to upload a file from any device
2. Users should be able to download a file from any device
3. Sync files across multiple devices 
4. Users should be able to share a file with other users and view a file shared with them
5. See file revisions 
6. Send notification when file is shared / edited / deleted 
7. ACID-ity
8. Offline editing 
9. Users should be able to edit files - Out of scope
10. Users should be able to view files without downloading them - Out of scope

### Non Functional Requirements
1. System should be highly available 
2. Support files as large as 50 GB
3. Secure and reliable - we should be able to recover files if they are lost / corrupted 
4. Low bandwidth usage - especially for mobile users 
5. Low latency - upload / download as fast as possible
6. Storage limit per users - Out of scope
7. Support file versioning - Out of scope 
8. Scan for viruses and malware - Out of scope

### Capacity Estimation
1. 50 GB file with 100Mbps would take 1.1 hours to upload
2. 50 mm users, 10 mm DAU
3. Users get 10GB free space
4. 2 file uploads / day, average file size 500Kb 
5. 1:1 read to write ratio
6. Total space - 500 Petabyte 
7. QPS for upload - 10 million * 2 uploads / 24 hours ~= 240
8. Peak QPS = 2 * 240 = 480

### Object Model
1. File - raw data user wants to upload 
2. File metadata - name, type, size, owner
3. User

### APIs
1. POST /files
	Request:
	```json
	{
		File,
		FileMetadata
	}
	```

2. GET /files/{fileId} -> File and FileMetadata

3. POST /files/{fileId}/share
	Request:
	```json 
	{
		User
	}
	```

4. File Metadata
	```json
	{
	  "id": "123",
	  "name": "file.txt",
	  "size": 1000,
	  "mimeType": "text/plain",
	  "uploadedBy": "user1",
	  "status": "uploading",
	  "chunks": [
	    {
	      "id": "chunk1",
	      "status": "uploaded"
	    },
	    {
	      "id": "chunk2",
	      "status": "uploading"
	    },
	    {
	      "id": "chunk3",
	      "status": "not-uploaded"
	    }
	  ]
	}
	```
### HLD

![[Pasted image 20240412230630.png]]

1. **Uploader**: This is the client that uploads the file. It could be a web browser, a mobile app, or a desktop app.
2. **Downloader**: This is the client that downloads the file. Of course, this can be the same client as the uploader, but it doesn't have to be.
3. **LB & API Gateway**: This is the load balancer and API Gateway that sits in front of our application servers. It's responsible for routing requests to the appropriate server and handling things like SSL termination, rate limiting, and request validation.
4. **File Service**: The file service is only responsible for writing to and from the file metadata db as well as requesting presigned URLs from S3. 
5. **File Metadata DB**: This is where we store metadata about the files.
6. **S3**: This is where the files are actually stored. 
7. **CDN**: This is a content delivery network that caches files close to the user to reduce latency.

![[Pasted image 20240412235158.png]]

### Database Choices
1. File - blob storage like Amazon S3
2. File metadata - loosely structured with few relations - can use DynamoDB, MongoDB, Cassandra. Could even use PostgreSQL - need sharding and hash-based partitioning. Overloaded partitions can be solved by consistent hashing
3. For sharing - you can fully normalize the data and create a separate table for shares. Make an entry here every time a user shares a file with another user

### Interesting Algorithms/Approaches
1. Client will chunk the file into 5-10Mb pieces and calculate a fingerprint for each chunk. It will also calculate a fingerprint for the entire file, this becomes the fileId.
2. Client will send a GET request to fetch the FileMetadata for the file with the given fileId (fingerprint) in order to see if it already exists - in which case, we can resume the upload.
3. If the file does not exist, the client will POST a request to /files/presigned-url to get a presigned URL for the file. The backend will save the file metadata in the FileMetadata table with a status of "uploading" and the chunks array will be a list of the chunk fingerprints with a status of "not-uploaded".
4. Client will then upload each chunk to S3 using the presigned URL. After each chunk is uploaded, S3 will send a message to our backend using S3 event notifications. Our backend will then update the chunks field in the FileMetadata table to mark the chunk as "uploaded".
5. Once all chunks in our chunks array are marked as "uploaded", the backend will update the FileMetadata table to mark the file as "uploaded".

### Scaling Challenges
1. How do you handle large files?
	1. Progress Indicator
	2. Resumable uploads 
	Use **chunking** 
2. How to speed up uploads/downloads
	1. CDNs
	2. Chunking and upload chunks in parallel - only modified chunks are uploaded
	3. Compression (Gzip) - works well for text file, not as much for media / images 

### Deep Dive Topics
1. Presigned URLs - provided by S3 to allow temporary access to private resources, generated with a specific expiration time, after which they become invalid, offering a secure way to share files without altering permissions. 
2. AWS MultiPart upload 
3. S3 event notifications 
4. Move infrequently used data to cold storage - Amazon S3 Glacier 
5. File security - S3 can encrypt the files
6. Merge conflicts - first version wins, second gets a conflict and needs to resolve before uploading 
7. Notifications - 
	1. Long polling (Dropbox)
	2. WebSockets - not useful since its bidirectional

### References
System Design Interview – An insider's guide, Second Edition: Step by Step Guide, Tips and 15 System Design Interview Questions with Detailed Solutions
https://www.hellointerview.com/learn/system-design/answer-keys/dropbox
https://www.educative.io/courses/grokking-the-system-design-interview/m22Gymjp4mG
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-google-docs