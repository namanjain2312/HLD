# UNIQUE ID GENERATOR



Your first thought might be to use a primary key with the auto_increment attribute in a traditional database. However, auto_increment does not work in a distributed environment because a single database server is not large enough and generating unique IDs across multiple databases with minimal delay is challenging.

Scope

Candidate: What are the characteristics of unique IDs?
Interviewer: IDs must be unique and sortable.
Candidate: For each new record, does ID increment by 1?
Interviewer: The ID increments by time but not necessarily only increments by 1. IDs created in the evening are larger than those created in the morning on the same day.
Candidate: Do IDs only contain numerical values?
Interviewer: Yes, that is correct.
Candidate: What is the ID length requirement?
Interviewer: IDs should fit into 64-bit.
Candidate: What is the scale of the system?
Interviewer: The system should be able to generate 10,000 IDs per second

Multiple options can be used to generate unique IDs in distributed systems. The options we considered are:
• Multi-master replication
• Universally unique identifier (UUID)
• Ticket server
• Twitter snowflake approach

Multi-master replication

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/EFyNPYUd8wDn3r2hkfIGG.png?ixlib=js-3.7.0 "image.png")

UUID
A UUID is another easy way to obtain unique IDs. UUID is a 128-bit number used to identify information in computer systems. UUID has a very low probability of getting collusion.
Quoted from Wikipedia, “after generating 1 billion UUIDs every second for approximately 100 years would the probability of creating a single duplicate reach 50%” [1].
Here is an example of UUID: 09c93e62-50b4-468d-bf8a-c07e1040bfb2. UUIDs can be generated independently without coordination between servers. Figure 7-3 presents the
UUIDs design

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/dZodrWiPvz5ysqVJxsGVc.png?ixlib=js-3.7.0 "image.png")

In this design, each web server contains an ID generator, and a web server is responsible for generating IDs independently.
Pros:
• Generating UUID is simple. No coordination between servers is needed so there will not be any synchronization issues.
• The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers.
Cons:
• IDs are 128 bits long, but our requirement is 64 bits.
• IDs do not go up with time.
• IDs could be non-numeric.

Ticket Server

Ticket servers are another interesting way to generate unique IDs.

Pros:
• Numeric IDs.
• It is easy to implement, and it works for small to medium-scale applications.
Cons:
• Single point of failure. Single ticket server means if the ticket server goes down, all systems that depend on it will face issues. To avoid a single point of failure, we can set up multiple ticket servers. However, this will introduce new challenges such as data
synchronization.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/juXiDZ_Xeiiy6RMWf_qdx.png?ixlib=js-3.7.0 "image.png")



Twitter snowflake approach



Approaches mentioned above give us some ideas about how different ID generation systems work. However, none of them meet our specific requirements; thus, we need another
approach. Twitter’s unique ID generation system called “snowflake” [3] is inspiring and can satisfy our requirements

Each section is explained below.
• Sign bit: 1 bit. It will always be 0. This is reserved for future uses. It can potentially be
used to distinguish between signed and unsigned numbers.
• Timestamp: 41 bits. Milliseconds since the epoch or custom epoch. We use Twitter
snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
• Datacenter ID: 5 bits, which gives us 2 ^ 5 = 32 datacenters.
• Machine ID: 5 bits, which gives us 2 ^ 5 = 32 machines per datacenter.
• Sequence number: 12 bits. For every ID generated on that machine/process, the sequence
number is incremented by 1. The number is reset to 0 every millisecond.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/C85QlvuGiaMu5HlBIOkGT.png?ixlib=js-3.7.0 "image.png")

Timestamp

The most important 41 bits make up the timestamp section. As timestamps grow with time, IDs are sortable by time. Figure 7-7 shows an example of how binary representation is converted to UTC. You can also convert UTC back to binary representation using a similar method.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/P7aGIaEHx0qNUW_ri0Jzk.png?ixlib=js-3.7.0 "image.png")

The maximum timestamp that can be represented in 41 bits is
2 ^ 41 - 1 = 2199023255551 milliseconds (ms), which gives us: ~ 69 years = 2199023255551 ms / 1000 seconds / 365 days / 24 hours/ 3600 seconds. This means the ID generator will work for 69 years and having a custom epoch time close to today’s date delays the overflow time. After 69 years, we will need a new epoch time or adopt other techniques to migrate IDs.
Sequence number
Sequence number is 12 bits, which give us 2 ^ 12 = 4096 combinations. This field is 0 unless more than one ID is generated in a millisecond on the same server. In theory, a machine can
support a maximum of 4096 new IDs per millisecond

Add On Points if Time Permits

1>Section length tuning. For example, fewer sequence numbers but more timestamp bits are effective for low concurrency and long-term applications.
2>High availability. Since an ID generator is a mission-critical system, it must be highly available



