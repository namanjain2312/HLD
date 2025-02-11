# Rate Limiter

Benefits of Rate Limiter

1>Prevent resource starvation caused by denial of service (Dos) : Dos attack prevention

2>Reducing Cost : If we used a third party app like send grid and if we send unlimited number of mails then cost would be high for us

3>Prevent servers from being overloaded

Scope

Candidate: What kind of rate limiter are we going to design? Is it a client-side rate limiter or server-side API rate limiter? Interviewer: Great question. We focus on the server-side API rate limiter.
Candidate: Does the rate limiter throttle API requests based on IP, the user ID, or other properties? Interviewer: The rate limiter should be flexible enough to support different sets of throttle rules.
Candidate: What is the scale of the system? Is it built for a startup or a big company with a large user base? Interviewer: The system must be able to handle a large number of requests.
Candidate: Will the system work in a distributed environment? Interviewer: Yes.
Candidate: Is the rate limiter a separate service or should it be implemented in application code? Interviewer: It is a design decision up to you.
Candidate: Do we need to inform users who are throttled? Interviewer: Yes



Requirements
Here is a summary of the requirements for the system:
• Accurately limit excessive requests.
• Low latency. The rate limiter should not slow down HTTP response time.
• Use as little memory as possible.
• Distributed rate limiting. The rate limiter can be shared across multiple servers or
processes.
• Exception handling. Show clear exceptions to users when their requests are throttled.
• High fault tolerance. If there are any problems with the rate limiter (for example, a cache
server goes offline), it does not affect the entire system.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/lZGhJzkfpPVz_O0ofgi_Q.png?ixlib=js-3.7.0 "image.png")



Algorithms For Rate Limiter 

1. Token Bucket
2. Leaking Bucket
3. Fixed Window Counter 
4. Sliding Window Log 
5. Sliding Window Counter
Token Bucket Algo



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/sKkpcJNRQ65CPXxQRBFEO.png?ixlib=js-3.7.0 "image.png")

Pros:
• The algorithm is easy to implement.
• Memory efficient.
• Token bucket allows a burst of traffic for short periods. A request can go through as long as there are tokens left.
Cons:
• Two parameters in the algorithm are bucket size and token refill rate. However, it might be challenging to tune them properly

Leaking Bucket Algorithm 

Implemented by FIFO queue 

Leaking bucket algorithm takes the following two parameters:
• Bucket size: it is equal to the queue size. The queue holds the requests to be processed at a fixed rate.
• Outflow rate: it defines how many requests can be processed at a fixed rate, usually in seconds. Shopify, an ecommerce company, uses leaky buckets for rate-limiting [7].
Pros:
• Memory efficient given the limited queue size.
• Requests are processed at a fixed rate therefore it is suitable for use cases that a stable outflow rate is needed.
Cons:
• A burst of traffic fills up the queue with old requests, and if they are not processed in time, recent requests will be rate limited.
• There are two parameters in the algorithm. It might not be easy to tune them properly



![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/QHLRcM5rNKZhLDaYJpc18.png?ixlib=js-3.7.0 "image.png")



Fixed window counter algorithm 

Fixed window counter algorithm works as follows: 

• The algorithm divides the timeline into fix-sized time windows and assign a counter for each window. 

• Each request increments the counter by one.

 • Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/g8xmwBtbi1gOft_4ZuhF1.png?ixlib=js-3.7.0 "image.png")

Pros:
• Memory efficient.
• Easy to understand.
• Resetting available quota at the end of a unit time window fits certain use cases.
Cons:
• Spike in traffic at the edges of a window could cause more requests than the allowed quota to go through.

Sliding window log algorithm 

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/_BLWJNv34IiYchihuQDR6.png?ixlib=js-3.7.0 "image.png")

Pros:
• Rate limiting implemented by this algorithm is very accurate. In any rolling window, requests will not exceed the rate limit.
Cons:
• The algorithm consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory.

Sliding Window Counter Algorithm 

The sliding window counter algorithm is a hybrid approach that combines the fixed window counter and sliding window log

Assume the rate limiter allows a maximum of 7 requests per minute, and there are 5 requests in the previous minute and 3 in the current minute. For a new request that arrives at a 30% position in the current minute, the number of requests in the rolling window is calculated using the following formula:
• Requests in current window + requests in the previous window * overlap percentage of the rolling window and previous window
• Using this formula, we get 3 + 5 * 0.7% = 6.5 request. Depending on the use case, the number can either be rounded up or down. In our example, it is rounded down to 6. Since the rate limiter allows a maximum of 7 requests per minute, the current request can go through. However, the limit will be reached after receiving one more request.

Pros
• It smooths out spikes in traffic because the rate is based on the average rate of the previous window.
• Memory efficient.
Cons
• It only works for not-so-strict look back window. It is an approximation of the actual rate because it assumes requests in the previous window are evenly distributed. However, this problem may not be as bad as it seems. According to experiments done by Cloudflare [10], only 0.003% of requests are wrongly allowed or rate limited among 400 million requests.

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/64FTkT6GIrzCgTiVVoCwK.png?ixlib=js-3.7.0 "image.png")

High-level architecture

Where shall we store counters? Using the database is not a good idea due to slowness of disk access. In-memory cache is chosen because it is fast and supports time-based expiration strategy. For instance, Redis [11] is a popular option to implement rate limiting. It is an in memory store that offers two commands: INCR and EXPIRE.
• INCR: It increases the stored counter by 1.
• EXPIRE: It sets a timeout for the counter. If the timeout expires, the counter is automatically deleted

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/dnt7qiN-gvFci9gXGESKz.png?ixlib=js-3.7.0 "image.png")

• The client sends a request to rate limiting middleware. 

• Rate limiting middleware fetches the counter from the corresponding bucket in Redis and checks if the limit is reached or not.

 • If the limit is reached, the request is rejected. 

• If the limit is not reached, the request is sent to API servers. Meanwhile, the system increments the counter and saves it back to Redis

Rate limiter headers
How does a client know whether it is being throttled? And how does a client know the number of allowed remaining requests before being throttled? The answer lies in HTTP response headers. The rate limiter returns the following HTTP headers to clients:
X-Ratelimit-Remaining: The remaining number of allowed requests within the window.
X-Ratelimit-Limit: It indicates how many calls the client can make per time window.
X-Ratelimit-Retry-After: The number of seconds to wait until you can make a request again without being throttled. When a user has sent too many requests, a 429 too many requests error and X-Ratelimit-Retry-After header are returned to the client

Components

• Rules are stored on the disk. Workers frequently pull rules from the disk and store them in the cache.
• When a client sends a request to the server, the request is sent to the rate limiter middleware first.
• Rate limiter middleware loads rules from the cache. It fetches counters and last request timestamp from Redis cache. Based on the response, the rate limiter decides:
• if the request is not rate limited, it is forwarded to API servers.
• if the request is rate limited, the rate limiter returns 429 too many requests error to the client. In the meantime, the request is either dropped or forwarded to the queue.

Rate limiter in a distributed environment
Building a rate limiter that works in a single server environment is not difficult. However, scaling the system to support multiple servers and concurrent threads is a different story.
There are two challenges:
• Race condition
• Synchronization issue

Race condition

 

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/v2PDy0uHBE2y_WGBbK0K3.png?ixlib=js-3.7.0 "image.png")

Locking could not be used as it would make redis slower

Two strategies are commonly used to solve the problem:

1. Lua script 
2. Sorted sets data structure in Redis 
Synchronization issue

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/P5Oiq6gXA7ppK0dEK5fHg.png?ixlib=js-3.7.0 "image.png")

Solution 

 Use centralized Redis 

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/1apEY_VzymFP_pQynJoxo.png?ixlib=js-3.7.0 "image.png")

Performance optimization

Multi-data center setup is crucial for a rate limiter because latency is high f or users located far away from the data center

Monitoring
After the rate limiter is put in place, it is important to gather analytics data to check whether the rate limiter is effective. Primarily, we want to make sure:
• The rate limiting algorithm is effective.
• The rate limiting rules are effective.
For example, if rate limiting rules are too strict, many valid requests are dropped. In this case, we want to relax the rules a little bit. In another example, we notice our rate limiter becomes ineffective when there is a sudden increase in traffic like flash sales. In this scenario, we may
replace the algorithm to support burst traffic. Token bucket is a good fit here.





