# Design Youtube

 Designing  YouTube is designing similar like designing Netflix

Scope of Problem 

1> what features are important ? ability to upload a video and watch a video

2>  question what clients do we need to support? mobile apps web browsers and smart TV

3>  how many daily active users do we have? 5 million

4> what is the average daily time spend on the product ?  30 minutes

5>  do we need to support International users ?  yes a lot percentage of users are International users

6>  what are the supported video resolutions ?  the system supports resolutions and formats

7>  question is encryption required ? yes

8>  any file size requirement for videos ?  all platform focuses on small and medium size videos the maximum video size is 1 GB

9>Can we leverage some of the existing cloud infrastructures provided by Amazon, Google, or Microsoft?n. Building everything from scratch is unrealistic for most companies, it is recommended to leverage some of the existing cloud services

We focus on designing a video streaming service with the following features:
• Ability to upload videos fast
• Smooth video streaming
• Ability to change video quality
• Low infrastructure cost
• High availability, scalability, and reliability requirements
• Clients supported: mobile apps, web browser, and smart TV

Back of the envelope estimation 

1. Assume the product has 5 million daily active users (DAU).
2. Users watch 5 videos per day 
3. 10 % of users upload 1 video per day 
4. Assume the average video size is 300 MB
5. Total daily storage space needed : 5 million *_ 10 % _ * 300 MB = 150 TB
6. CDN cost
    1. When cloud CDN serves a video , you are charged for data transferred out of the CDN
    2. Lets use Amazon CDN cloud front for cost estimation , assume 100 % traffic is served from US , average cost per GB is $0.02 per GB.
    3. Video Streaming cost  : 5million * 5 videos * 0.3 GB * $0.02 = $150,000 per day .
7. So from the rough cost estimation we figured out that CDN cost would be a lot .


Two types of flow in this 

1. Video Uploading Flow 
2. Video Streaming Flow


Now how video uploading flow works  : 

1. Upload the actual video (Part-1)
2. Update video metadata : Meta data contains the video url , size , resolution , format , user info etc . (Part-2)
Components in Video Upload Flow  (Part-1)

1. User :  A computer , mobile phone or smart tv
2. Load Balancer : Evenly distributes requests among api servers 
3. API Servers : All request go through API servers except videos streaming .Everything except video streaming goes through API servers . This Includes feed recommendation , generating video upload URL , updating meta data db and cache , user signup etc.
4. Metadata DB : Video meta data are stored in metadata db . It is sharded and replicated to meet performance and high availability requirement.
5. MetaData Cache : For better performance , video meta data and user objects are cached .
6. Original Storage  : A blob storage system is used to store original videos . Blob storage - a binary large object is a collection of binary data stored as single entity in a database management system .
7. Transcoding Servers : It is a blob storage that stores transcoded video files 
8. CDN : Videos are cached in CDN . When you press play , a video is streamed from the CDN .
9. Completion Queue : It is a message queue that stores information about video transcoding completion events 
10. Compeletion Handleer : This consists of a list of workers that pull event data from the completion queue and update the metadata cache and databse 
Update the meta data (Part-2)

1. When a file is been uploaded to the original storage , the client in parrarel sends a request to update the video metadata , including file name , size , format etc. API servers update the metadata cache and database.
 

Video Streaming Flow

Streaming Protocols 

1. MPEG-DASH , MPEG stands for Moving Picture Experts Group and DASH stands for Dynamic Adaptive Streaming over HTTP
2. Apple HLS , HLS stands for HTTP Live Streaming 
3. Microsoft Smooth Streaming
4. Adobe HTTP Dynamic Streaming (HDS)
Videos are streamed directly from CDN , the edge server closed from you will deliever the video thus very little latency .

Video Transcoding 

1. What is video transcoding 
    1.  When we record a video , the device gives a video in certain format , if i wnat to play this video smoothly on other device the video must be encoded into compatible bitrates and formats . 
    2. Bitrates is the rate at which bits are processed over time , 
    3. Higher bittrate generally means higer video quality 
    4. Higher bitrate streams need more processing power and fast internet.
2. Advatage of Video Transcoding 
    1. Raw video consumes large amount of storeage space /
    2. Many devices or browser supports only cretain types of video format thus encoding video provided compatibility 
    3. To ensure high quality video while maitninf smooth playback its good idea is to send the higher resolution to those user which have higher bandwidth abd lower resolution to user having lower bandwidht 
    4. Network condition can change , to ensure smooth playback , swithcing video quality automatically based on network condiiton is essential 
3.  Endcoding formats parts
    1. Contianer : This is like basket that contians the video file , audio and metdata . You can tell teh container format by the file extensions such .avi.mov or .mp4
    2. Codecs : These are compression and decompression algortihms aim to reducre the video sizre while preserving the video quality . The mose used video codeces are H.264,VP9 and HEVC
## Video Transcoding Architecture 


![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/cgbBbrXOGb7Lj4pu9gc_i.png?ixlib=js-3.7.0 "image.png")

1. Preprocesser The preprocessor has 4 responsibilities:
    1. Video splitting. Video stream is split or further split into smaller Group of Pictures (GOP) alignment. GOP is a group/chunk of frames arranged in a specific order. Each chunk is an independently playable unit, usually a few seconds in length.
    2. Some old mobile devices or browsers might not support vides splitting. Preprocessor split videos by GOP alignment for old clients.
    3. DAG generation. The processor generates DAG based on configuration files client programmers write.
    4. Cache data. The preprocessor is a cache for segmented videos. For better reliability, the preprocessor stores GOPs and metadata in temporary storage. If video encoding fails, the system could use persisted data for retry operations.
2. DAG scheduler ( Directed acyclic graph model (DAG)) : The DAG scheduler splits a DAG graph into stages of tasks and puts them in the task queue in the resource manager


![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/p--9CJSlu0P_wHfuPCJZ8.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/7kkFOUD30dtnMp4MHnuNM.png?ixlib=js-3.7.0 "image.png")

3>Resource manager : The resource manager is responsible for managing the efficiency of resource allocation. It contains 3 queues and a task scheduler .
• Task queue: It is a priority queue that contains tasks to be executed.
• Worker queue: It is a priority queue that contains worker utilization info.
• Running queue: It contains info about the currently running tasks and workers running the tasks.
• Task scheduler: It picks the optimal task/worker, and instructs the chosen task worker to execute the job

The resource manager works as follows:
• The task scheduler gets the highest priority task from the task queue.
• The task scheduler gets the optimal task worker to run the task from the worker queue.
• The task scheduler instructs the chosen task worker to run the task.
• The task scheduler binds the task/worker info and puts it in the running queue.
• The task scheduler removes the job from the running queue once the job is done.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/rmIEsgnp9fRLerOIscQe6.png?ixlib=js-3.7.0 "image.png")

4>Task workers : Task workers run the tasks which are defined in the DAG

5>Temporary storage : Multiple storage systems are used here. The choice of storage system depends on factors like data type, data size, access frequency, data life span, etc.

6>Encoded video : Encoded video is the final output of the encoding pipeline.

System Optimization 

1>Speed Optimization : Parallelize Video Uploading 

Break original video in multiple GOP on client end while uploading for faster speed and also to allow to continue from previous in case of network break



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/EJno98u7JNDsnL-s7NVBM.png?ixlib=js-3.7.0 "image.png")

2>Speed Optimization : Place upload centre close to users.

Using cdn as upload centre and setting multiple upload centre across the globe 

3>Speed Optimization : Parallelism everywhere 

Decouple the system as much as possible 

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/SkiFG7ezLbxIcuzdp9D3i.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/AkXxXLIO3BfaQQIJfXtBE.png?ixlib=js-3.7.0 "image.png")

4>Safety Optimization : pre signed upload url 

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/bye33smnBY6gJd2REkxal.png?ixlib=js-3.7.0 "image.png")

Without pre signed url access to upload is not given

5>Safety Optimization : Protect you video

a>DRM system 

b>AES encryption on video so only authorized user can watch 

c>Visual Watermarking 

5>Cost Saving Optimization

a>Only server the most popular videos from CDN and other from our high capacity storage video servers as CDN are expensive

b>For less popular content , we may not need to store many encoded video version . Short video can be encoded on demand 

c>Some videos are popular only in certain region , no need to distribute these video to other region 

d>Build your own cdn



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/lr8K_9haWvO174a9liZBb.png?ixlib=js-3.7.0 "image.png")

Error Handling 

Two types of errors
exist:
• Recoverable error. For recoverable errors such as video segment fails to transcode, the general idea is to retry the operation a few times. If the task continues to fail and the system believes it is not recoverable, it returns a proper error code to the client.
• Non-recoverable error. For non-recoverable errors such as malformed video format, the system stops the running tasks associated with the video and returns the proper error code to the client.
Typical errors for each system component are covered by the following playbook:
• Upload error: retry a few times 

Split video error: if older versions of clients cannot split videos by GOP alignment, the entire video is passed to the server. The job of splitting videos is done on the server-side.
• Transcoding error: retry.
• Preprocessor error: regenerate DAG diagram.
• DAG scheduler error: reschedule a task.
• Resource manager queue down: use a replica.
• Task worker down: retry the task on a new worker.
• API server down: API servers are stateless so requests will be directed to a different API server.
• Metadata cache server down: data is replicated multiple times. If one node goes down , you can still access other nodes to fetch data. We can bring up a new cache server to replace the dead one.
• Metadata DB server down:
• Master is down. If the master is down, promote one of the slaves to act as the new master.
• Slave is down. If a slave goes down you can use another slave for reads and bring up another database server to replace the dead one



