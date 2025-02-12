# KeyValue Store

Scope

• The size of a key-value pair is small: less than 10 KB.
• Ability to store big data.
• High availability: The system responds quickly, even during failures.
• High scalability: The system can be scaled to support large data set.
• Automatic scaling: The addition/deletion of servers should be automatic based on traffic.
• Tunable consistency.
• Low latency.

We will have to know about CAP theorm first 





System components

In this section, we will discuss the following core components and techniques used to build a key-value store:
• Data partition : Use consistent hashing
• Data replication : Use different server for each node in that hash ring 
• Consistency : 
• Inconsistency resolution
• Handling failures
• System architecture diagram
• Write path
• Read path



Since data is replicated at multiple nodes, it must be synchronized across replicas. Quorum
consensus can guarantee consistency for both read and write operations. Let us establish a
few definitions first.
N = The number of replicas
W = A write quorum of size W. For a write operation to be considered as successful, write
operation must be acknowledged from W replicas.
R = A read quorum of size R. For a read operation to be considered as successful, read
operation must wait for responses from at least R replicas.
Consider the following example shown in Figure 6-6 with N = 3



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/4h5-O70fBpXAea919dOhT.png?ixlib=js-3.7.0 "image.png")

If W + R > N, strong consistency is guaranteed because there must be at least one
overlapping node that has the latest data to ensure consistency.
How to configure N, W, and R to fit our use cases? Here are some of the possible setups:
If R = 1 and W = N, the system is optimized for a fast read.
If W = 1 and R = N, the system is optimized for fast write.
If W + R > N, strong consistency is guaranteed (Usually N = 3, W = R = 2).
If W + R <= N, strong consistency is not guaranteed.
Depending on the requirement, we can tune the values of W, R, N to achieve the desired level
of consistency.

Consistency models

• Strong consistency: any read operation returns a value corresponding to the result of the
most updated write data item. A client never sees out-of-date data.
• Weak consistency: subsequent read operations may not see the most updated value.
• Eventual consistency: this is a specific form of weak consistency. Given enough time, all
updates are propagated, and all replicas are consistent.

Dynamo and Cassandra adopt eventual consistency

Inconsistency resolution: versioning



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/Gry1Cr6oq07ykKu-tydR_.png?ixlib=js-3.7.0 "image.png")



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/QsInxVp2QXgQBVeJv-rRC.png?ixlib=js-3.7.0 "image.png")

A vector clock is a [server, version] pair associated with a data item. It can be used to check if one version precedes, succeeds, or in conflict with others.



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/v4wCn8R0ZGKvEirNYB4Dl.png?ixlib=js-3.7.0 "image.png")

Assume a vector clock is represented by D([S1, v1], [S2, v2], …, [Sn, vn]), where D is a data item, v1 is a version counter, and s1 is a server number, etc. If data item D is written to server Si, the system must perform one of the following tasks. • Increment vi if [Si, vi] exists. • Otherwise, create a new entry [Si, 1]. The above abstract logic is explained with a concrete example as shown in Figure 6-9.

1. A client writes a data item D1 to the system, and the write is handled by server Sx,
which now has the vector clock D1[(Sx, 1)].
2. Another client reads the latest D1, updates it to D2, and writes it back. D2 descends
from D1 so it overwrites D1. Assume the write is handled by the same server Sx, which
now has vector clock D2([Sx, 2]).
3. Another client reads the latest D2, updates it to D3, and writes it back. Assume the write
is handled by server Sy, which now has vector clock D3([Sx, 2], [Sy, 1])).
4. Another client reads the latest D2, updates it to D4, and writes it back. Assume the write
is handled by server Sz, which now has D4([Sx, 2], [Sz, 1])).
5. When another client reads D3 and D4, it discovers a conflict, which is caused by data
item D2 being modified by both Sy and Sz. The conflict is resolved by the client and
updated data is sent to the server. Assume the write is handled by Sx, which now has
D5([Sx, 3], [Sy, 1], [Sz, 1]). We will explain how to detect conflict shortly.
Using vector clocks, it is easy to tell that a version X is an ancestor (i.e. no conflict) of
version Y if the version counters for each participant in the vector clock of Y is greater than or
equal to the ones in version X. For example, the vector clock D([s0, 1], [s1, 1])] is an
ancestor of D([s0, 1], [s1, 2]). Therefore, no conflict is recorded.
Two notable downsides of above

1. First, vector clocks add complexity to the client because it needs to implement conflict resolution logic.
2. Second, the [server: version] pairs in the vector clock could grow rapidly. To fix this
problem, we set a threshold for the length, and if it exceeds the limit, the oldest pairs are
removed. This can lead to inefficiencies in reconciliation because the descendant relationship
cannot be determined accurately. However, based on Dynamo paper [4], Amazon has not yet
encountered this problem in production; therefore, it is probably an acceptable solution for
most companies.
Handling failures

1>Failure detection - In a distributed system, it is insufficient to believe that a server is down because another server says so. Usually, it requires at least two independent sources of information to mark a server down.

1. all-to-all multicasting is a straightforward solution where each serer is been connceted to all other , but its inefficient .
2. 


![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/bug8bcfmgrgtFBg89Qwje.png?ixlib=js-3.7.0 "image.png")

c.Gossip Protocol 

A better solution is to use decentralized failure detection methods like gossip protocol.
Gossip protocol works as follows:
• Each node maintains a node membership list, which contains member IDs and heartbeat
counters.
• Each node periodically increments its heartbeat counter.
• Each node periodically sends heartbeats to a set of random nodes, which in turn
propagate to another set of nodes.
• Once nodes receive heartbeats, membership list is updated to the latest info.
• If the heartbeat has not increased for more than predefined periods, the member is
considered as offline

• Node s0 maintains a node membership list shown on the left side.
• Node s0 notices that node s2’s (member ID = 2) heartbeat counter has not increased for a
long time.
• Node s0 sends heartbeats that include s2’s info to a set of random nodes. Once other
nodes confirm that s2’s heartbeat counter has not been updated for a long time, node s2 is
marked down, and this information is propagated to other nodes.



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/0zcP0KohnfS_X2wlWWzPl.png?ixlib=js-3.7.0 "image.png")

2>Handling temporary failures 

After failures have been detected through the gossip protocol, the system needs to deploy
certain mechanisms to ensure availability. In the strict quorum approach, read and write
operations could be blocked as illustrated in the quorum consensus section.
A technique called “sloppy quorum” [4] is used to improve availability. Instead of enforcing
the quorum requirement, the system chooses the first W healthy servers for writes and first R
healthy servers for reads on the hash ring. Offline servers are ignored.
If a server is unavailable due to network or server failures, another server will process
requests temporarily. When the down server is up, changes will be pushed back to achieve
data consistency. This process is called hinted handoff. Since s2 is unavailable in Figure 6-
12, reads and writes will be handled by s3 temporarily. When s2 comes back online, s3 will
hand the data back to s2.



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/_a_9aluipSHbS9YRgkDJ3.png?ixlib=js-3.7.0 "image.png")

Handling permanent failures

Refer the alexu part-1 book pg 101

Handling data center outage

Data center outage could happen due to power outage, network outage, natural disaster, etc. To build a system capable of handling data center outage, it is important to replicate data across multiple data centers. Even if a data center is completely offline, users can still access data through the other data centers.

Main features of the architecture are listed as follows:
• Clients communicate with the key-value store through simple APIs: get(key) and put(key,
value).
• A coordinator is a node that acts as a proxy between the client and the key-value store.
• Nodes are distributed on a ring using consistent hashing.
• The system is completely decentralized so adding and moving nodes can be automatic.
• Data is replicated at multiple nodes.
• There is no single point of failure as every node has the same set of responsibilities.
As the design is decentralized, each node performs many tasks as presented in

Write path

1. The write request is persisted on a commit log file.
2. Data is saved in the memory cache.
3. When the memory cache is full or reaches a predefined threshold, data is flushed to
SSTable [9] on disk. Note: A sorted-string table (SSTable) is a sorted list of <key, value>
pairs. For readers interested in learning more about SStable, refer to the reference material
Read path

1. The system first checks if data is in memory. If not, go to step 2.
2. If data is not in memory, the system checks the bloom filter.
3. The bloom filter is used to figure out which SSTables might contain the key.
4. SSTables return the result of the data set.
5. The result of the data set is returned to the client.


![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/BsLHH7dOgRlt733VAie_Y.png?ixlib=js-3.7.0 "image.png")



