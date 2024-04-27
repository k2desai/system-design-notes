1. Very good for applications that require rapid and frequent read and write operations 
2. Uses reader-follower protocol - making it possible to use replicas for heavy reading 
3. MongoDB ensures atomicity in concurrent write operations and avoids collisions by returning duplicate-key errors for record-duplication issues.
4. Good when you dont need complex joins in your data
https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/design-and-deployment-of-tinyurl#Components