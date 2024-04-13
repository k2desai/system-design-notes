### Presigned URLs
Presigned URLs are a feature provided by cloud storage services, such as Amazon S3, that allow temporary access to private resources. These URLs are generated with a specific expiration time, after which they become invalid, offering a secure way to share files without altering permissions. When a presigned URL is created, it includes authentication information as part of the query string, enabling controlled access to otherwise private objects.

This makes them ideal for use cases like temporary file sharing, uploading objects to a bucket without giving users full API access, or providing limited-time access to resources. Presigned URLs can be generated programmatically using the cloud provider's SDK, allowing developers to integrate this functionality into applications seamlessly. This method enhances security by ensuring that sensitive data remains protected while still being accessible to authorized users for a limited period.

### Event Notifications 


### MultiPart upload

### S3 Glacier