# WebCrawler

Scope 

The basic algorithm of a web crawler is simple: 1. Given a set of URLs, download all the web pages addressed by the URLs. 2. Extract URLs from these web pages 3. Add new URLs to the list of URLs to be downloaded. Repeat these 3 steps

Candidate: What is the main purpose of the crawler? Is it used for search engine indexing,
data mining, or something else?
Interviewer: Search engine indexing.
Candidate: How many web pages does the web crawler collect per month?
Interviewer: 1 billion pages.
Candidate: What content types are included? HTML only or other content types such as
PDFs and images as well?
Interviewer: HTML only.
Candidate: Shall we consider newly added or edited web pages?
Interviewer: Yes, we should consider the newly added or edited web pages.
Candidate: Do we need to store HTML pages crawled from the web?
Interviewer: Yes, up to 5 years
Candidate: How do we handle web pages with duplicate content?
Interviewer: Pages with duplicate content should be ignored.





![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/0VvFpFId5kPxHlCZJkPeM.png?ixlib=js-3.7.0 "image.png")

DFS vs BFS 

Use BFS

Avoid Impolite Behaviour

URL frontier
URL frontier helps to address these problems. A URL frontier is a data structure that stores
URLs to be downloaded. The URL frontier is an important component to ensure politeness,
URL prioritization, and freshness. A few noteworthy papers on URL frontier are mentioned
in the reference materials [5] [9]. The findings from these papers are as follo

![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/LSmnf7g8ykvUDIGba8hAt.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/BKBDtHn6WJpwXZWKs1suy.png?ixlib=js-3.7.0 "image.png")



![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/C5qC-wDP93Hi64UdeXFC8.png?ixlib=js-3.7.0 "image.png")

HTML Downloader
The HTML Downloader downloads web pages from the internet using the HTTP protocol.
Before discussing the HTML Downloader, we look at Robots Exclusion Protocol first.
Robots.txt
Robots.txt, called Robots Exclusion Protocol, is a standard used by websites to communicate
with crawlers. It specifies what pages crawlers are allowed to download. Before attempting to
crawl a web site, a crawler should check its corresponding robots.txt first and follow its rul



1. Distributed crawl 

![image.png](https://eraser.imgix.net/workspaces/ICDAvN3jM6gR93bBUxPZ/pqzq4S07fqcma47xGOPbSlv9Jtt1/qc-DEkllnYkuX9h8K2vcW.png?ixlib=js-3.7.0 "image.png")

Cache DNS Resolver
DNS Resolver is a bottleneck for crawlers because DNS requests might take time due to the
synchronous nature of many DNS interfaces. DNS response time ranges from 10ms to
200ms. Once a request to DNS is carried out by a crawler thread, other threads are blocked
until the first request is completed. Maintaining our DNS cache to avoid calling DNS
frequently is an effective technique for speed optimization. Our DNS cache keeps the domain
name to IP address mapping and is updated periodically by cron jobs.

Locality
Distribute crawl servers geographically. When crawl servers are closer to website hosts,
crawlers experience faster download time. Design locality applies to most of the system
components: crawl servers, cache, queue, storage, etc.

Short timeout
Some web servers respond slowly or may not respond at all. To avoid long wait time, a
maximal wait time is specified. If a host does not respond within a predefined time, the
crawler will stop the job and crawl some other pages.



Detect and avoid problematic content
This section discusses the detection and prevention of redundant, meaningless, or harmful
content.

1. Redundant content
As discussed previously, nearly 30% of the web pages are duplicates. Hashes or checksums
help to detect duplication [11].
2. Spider traps
A spider trap is a web page that causes a crawler in an infinite loop. For instance, an infinite
deep directory structure is listed as follows:
[﻿www.spidertrapexample.com/foo/bar/foo/bar/foo/bar/…](http://www.spidertrapexample.com/foo/bar/foo/bar/foo/bar/%E2%80%A6) 
Such spider traps can be avoided by setting a maximal length for URLs. However, no onesize-fits-all solution exists to detect spider traps. Websites containing spider traps are easy to
identify due to an unusually large number of web pages discovered on such websites. It is
hard to develop automatic algorithms to avoid spider traps; however, a user can manually
3. verify and identify a spider trap, and either exclude those websites from the crawler or apply
some customized URL filters.
3. Data noise
Some of the contents have little or no value, such as advertisements, code snippets, spam
URLs, etc. Those contents are not useful for crawlers and should be excluded if possible.


