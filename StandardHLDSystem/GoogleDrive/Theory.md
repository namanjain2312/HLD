# Google Drive

Scope 

1. What are the most important feature ?
Ans : Upload and download file , file sync and notifications

2>Is it a mobile app , we app or both ?

Ans : Both 

3>Do files need to be encrypted ?

Ans : Yes , files in the storage must be encrypted 

4>Is there a file size limit ?

Ans : Yes , files must be 10 GB or smaller .

5>How many users does the product have 

Ans : 10 M DAU

Functional Requirement 

1>Add files , via drag and drop on  google drive 

2>Download files

3>Sync files , across multiple devices (when a file is added)

4>See file revisions

5>Share files with your family , friends and coworkers

6>Send notification when a file is edited , deleted or shared 

Non Functional Requirement

1>Reliability : System should be extremely reliable , data loss is unacceptable 

2>Fast Sync Speed

3>Bandwidth usage , if product takes lots of bandwidth user would be unhappy specially those on mobile data plan

4>Scalable

5>High Availability

Back Of the Envelope Estimation 

1>Assume the application has 50 million signed up users and 10 million DAU

2>Users get 10 GB free space 

3>Assume users upload 2 files per day , the average file size is 500 KB

4>1:1 read to write ratio

5>Total space allowed 50 million * 10 GB = 500 Petabyte

6>QPS for upload API : 10 million * 2 uploads  * / 24 hours / 3600 seconds = ~240 

7>Peak QPS = QPS * 2 = 480

Basic HLD

Start with simple approach 

1>Use a single server 

2>Upload / Download / Get File Revision History  API

3>Db to store file 

4>DB to store metadata



Sync Conflicts

1. To handle sync commit Let say two user push data users whose data would be pushed and accepted first would be shown and updated , user 2 data would be rejected and thus for user 2 We could show both the original updated file and local file of user 2 and user 2 can commit and merge the changes .
Components

1. Use Amazon S3 Bucket for file storage , as it provides same - region and cross region replication , redundant files stored in multiple regions to guard against data loss and ensure availability .Ex : Same region replication - Bucket A in region X , replicated Bucket A to Bucket B and C in region Y , Z respectively 
2. Load Balancer to distribute the traffic 
3. Web servers where my application would be hosted 
4. Metadata database in which also do proper sharding .
5. Block Servers : Block servers upload blocks to cloud storage , a file is split in many blocks and each with unique hash value , stored in our metadata database , each block is treated as independent block and stored in our storage system , to reconstruct the file blocks are joined in particular order .
6. Cold Storage : Cold Storage is system design for storing inactive data , meaning file are not accessed for a long time .
7. API servers : Responsible for everything other than the uploading flow , API servers are used for user authentication , managing user profile , updating file in metadata etc.
8. Metadata Database : It stores meta data of users , file , blocks , version etc. Please note that files are stored in cloud and the metadata db contains only metadata 
9. Metadata Cache : Some of metadata is been cached for fast retrieval
10. Notification Service : Notifies client that the file is added / updated/ removed so they can pull the lates change 
11. Offline Back Up queue : If a client is offline and cannot pull the latest file changes , the offline queue store the  info so changes will be synced when the client is online .


Block Servers 

For large files that are updated regularly sending whole file again take a huge bandwidth . Two optimization :

1. Delta Sync : When the file is updated only modified blocks are synced instead of whole file (generally used when a file is modified )
2. Compression : Applying compression on block can significantly reduce the data size using tools like gzip etc (Generally used when a file is created )
High Consistency Requirement 

1. Our data should be strongly consistent and should follow ACID properties and for that to implement we have to use SQL database as they have this feature inbuild whereas sql do not have , so for synchronization logic and meta data logic sql / rdbms db should  be used 
Notifications

1. Long Polling - Drop Box used it
2. WebSocket 
We use long polling due to two reason 

1. Communication here is not bi directional 
2. Websocket are suited for bidirectional and also notification here are send infrequently with no burst of data
Save Storage Space 

To support file version history and ensure reliability , multiple version  of the same file is been stored across multiple data center . So storage space can be filled up quickly . THree technique to reduce the storage cost 

1. Deduplicate data blocks : Eliminating redundant blocks at the account level .
2. Enhanced Data backup strategy 
    1. Set A Limit  : We can set limit for a number of the version to store . If limit is reached the oldest version will be replaced with the new version 
    2. Keep Valubale Version :  Some files have been edited frequently , yo avoid unnecssary copes we could limit the number of version and weight more to the recent version 
    3. Moving infrequently used data to cold storage . Cold data is the data which have been not active for months or year , cold storage like Amazon S3 glacier is much cheaper than S3.


![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/lz7_UfHYF60x8hdJlywBg.png?ixlib=js-3.7.0 "image.png")



![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/lxJvzes7RO3bPE5eQ5eAK.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/SaBlxdlo6kAxbjF-XuMEW.png?ixlib=js-3.7.0 "image.png")



![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/TTCTxEwsvvXxRj7NCCAIu.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/Va8uSLuKVFiiCBw1X4RvM.png?ixlib=js-3.7.0 "image.png")



# Upload flow
![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/tnXoJBx_zuk0X1BsNsfnS.png?ixlib=js-3.7.0 "image.png")

# Download flow 


![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/Hnd8SV6SKg88xgyYbKlHH.png?ixlib=js-3.7.0 "image.png")



