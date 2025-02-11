# SearchAutoComplete System

## Scope
Candidate: Is the matching only supported at the beginning of a search query or in the
middle as well?
Interviewer: Only at the beginning of a search query.
Candidate: How many autocomplete suggestions should the system return?
Interviewer: 5
Candidate: How does the system know which 5 suggestions to return?
Interviewer: This is determined by popularity, decided by the historical query frequency.
Candidate: Does the system support spell check?
Interviewer: No, spell check or autocorrect is not supported.
Candidate: Are search queries in English?
Interviewer: Yes. If time allows at the end, we can discuss multi-language support.
Candidate: Do we allow capitalization and special characters?
Interviewer: No, we assume all search queries have lowercase alphabetic characters.
Candidate: How many users use the product?
Interviewer: 10 million DAU

## Requirements
Here is a summary of the requirements:
• Fast response time: As a user types a search query, autocomplete suggestions must show
up fast enough. An article about Facebook’s autocomplete system [1] reveals that the
system needs to return results within 100 milliseconds. Otherwise it will cause stuttering.
• Relevant: Autocomplete suggestions should be relevant to the search term.
• Sorted: Results returned by the system must be sorted by popularity or other ranking
models.
• Scalable: The system can handle high traffic volume.
• Highly available: The system should remain available and accessible when part of the
system is offline, slows down, or experiences unexpected network errors

Back of the envelope estimation
• Assume 10 million daily active users (DAU).
• An average person performs 10 searches per day.
• 20 bytes of data per query string:
• Assume we use ASCII character encoding. 1 character = 1 byte
• Assume a query contains 4 words, and each word contains 5 characters on average.
• That is 4 x 5 = 20 bytes per query.
• For every character entered into the search box, a client sends a request to the backend
for autocomplete suggestions. On average, 20 requests are sent for each search query. For
example, the following 6 requests are sent to the backend by the time you finish typing
“dinner”.
search?q=d

search?q=di
search?q=din
search?q=dinn
search?q=dinne
search?q=dinner
• ~24,000 query per second (QPS) = 10,000,000 users * 10 queries / day * 20 characters /
24 hours / 3600 seconds.
• Peak QPS = QPS * 2 = ~48,000
• Assume 20% of the daily queries are new. 10 million * 10 queries / day * 20 byte per
query * 20% = 0.4 GB. This means 0.4GB of new data is added to storage daily.



Now whole design consist of two parts 

1>Data Gathering Service

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/FLbBZUHx6BHaK9brJ2VCX.png?ixlib=js-3.7.0 "image.png")



2>Query Service 



![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/zNHIaduKcxrZjEHdqP0Dy.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/c6eUVAml9YY4q7g0omlGW.png?ixlib=js-3.7.0 "image.png")

Now remeber that we cannot use simple like query which we used to use in sql as it will not work here as datasize is huge 



Now for querying we use trie 

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/WEQ218rMbPf3CbNIJiw-A.png?ixlib=js-3.7.0 "image.png")

O(p) + O(c) + O(clogc)  

P = lenght of prefix

C= no of child 

clogc = sort c child and return top 5 child

Two optimization

1. Limit the max length of a prefix 
    1. P is relatively small and restrict its lenght so O(p)=O(1)
2. Cache top search queries at each node
    1. O(c) and O(clogc) = 1
So net time to return best 5 result would be ~ O(1)



![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/4sXTG-MAgW84aois4LhQf.png?ixlib=js-3.7.0 "image.png")



## Data Gathering Service 
1. Analytics Logs. It stores raw data about search queries. Logs are append-only and are not indexed.
    1. 


![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/lv3sKRGjh0pD68G_1McKt.png?ixlib=js-3.7.0 "image.png")

2>Aggregators. The size of analytics logs is usually very large, and data is not in the right format. We need to aggregate data so it can be easily processed by our system.

3>Aggregated Data. Table 13-4 shows an example of aggregated weekly data. “time” field represents the start time of a week. “frequency” field is the sum of the occurrences for the corresponding query in that week

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/_SzcjSMzfDyFE6sAadcK6.png?ixlib=js-3.7.0 "image.png")

4>Workers. Workers are a set of servers that perform asynchronous jobs at regular intervals. They build the trie data structure and store it in Trie DB.
5>Trie Cache. Trie Cache is a distributed cache system that keeps trie in memory for fast read. It takes a weekly snapshot of the DB.
6>Trie DB. Trie DB is the persistent storage. Two options are available to store the data:

1. Document store: Since a new trie is built weekly, we can periodically take a snapshot of it,
serialize it, and store the serialized data in the database. Document stores like MongoDB [4]
are good fits for serialized data.
2. Key-value store: A trie can be represented in a hash table form [4] by applying the
following logic:
• Every prefix in the trie is mapped to a key in a hash table.
• Data on each trie node is mapped to a value in a hash table.
## Query service 
Query Service optimizations

1. AJAX request. For web applications, browsers usually send AJAX requests to fetch autocomplete results. The main benefit of AJAX is that sending/receiving a request/response does not refresh the whole web page.
2. Browser caching : storing search results in browser cache for some time
3. Data sampling: For a large-scale system, logging every search query requires a lot of power and storage. Data sampling is important. For instance, only 1 out of every N requests is logged by the system


Filter Layer 

1. We have to remove hateful, violent, sexually explicit, or dangerous autocomplete suggestions. We add a filter layer (Figure 13-14) in front of the Trie Cache to filter out unwanted suggestions.


![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/v9ZJbtMioWsRODPo5UbKk.png?ixlib=js-3.7.0 "image.png")



## Scale the storage 
1. Since English is the only supported language, a naive way to shard is based on the first
character. Here are some examples.
• If we need two servers for storage, we can store queries starting with ‘a’ to ‘m’ on the
first server, and ‘n’ to ‘z’ on the second server.
• If we need three servers, we can split queries into ‘a’ to ‘i’, ‘j’ to ‘r’ and ‘s’ to ‘z’
2. Following above logic 26 server for 26 alphabet and similarly could be done level1 level2 etc , but the issue is that 'sa' starting word are more in combined with word 'x' ,'y','z' etc and thus inequal distribution server wise .
3. 


![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/aSIMqfmr59yTctxDiv9X3.png?ixlib=js-3.7.0 "image.png")



