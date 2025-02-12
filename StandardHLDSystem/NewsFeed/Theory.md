# NewsFeed

Scope

Candidate: Is this a mobile app? Or a web app? Or both?
Interviewer: Both
Candidate: What are the important features?
Interview: A user can publish a post and see her friends’ posts on the news feed page.
Candidate: Is the news feed sorted by reverse chronological order or any particular order such as topic scores? For instance, posts from your close friends have higher scores.
Interviewer: To keep things simple, let us assume the feed is sorted by reverse chronological order.
Candidate: How many friends can a user have?
Interviewer: 5000
Candidate: What is the traffic volume?
Interviewer: 10 million DAU
Candidate: Can feed contain images, videos, or just text?
Interviewer: It can contain media files, including both images and videos.



The design is divided into two flows: feed publishing and news feed building.
• Feed publishing: when a user publishes a post, corresponding data is written into cache
and database. A post is populated to her friends’ news feed.
• Newsfeed building: for simplicity, let us assume the news feed is built by aggregating
friends’ posts in reverse chronological order

Newsfeed API

1. Feed publishing API
2. Newsfeed retrieval API
Feed publishing

• User: a user can view news feeds on a browser or mobile app. A user makes a post with
content “Hello” through API:
/v1/me/feed?content=Hello&auth_token={auth_token}
• Load balancer: distribute traffic to web servers.
• Post service: persist post in the database and cache.

Web servers
Besides communicating with clients, web servers enforce authentication and rate-limiting.
Only users signed in with valid auth_token are allowed to make posts. The system limits the
number of posts a user can make within a certain period, vital to prevent spam and abusive
content.
Fanout service
Fanout is the process of delivering a post to all friends. Two types of fanout models are:
fanout on write (also called push model) and fanout on read (also called pull model). Both
models have pros and cons. We explain their workflows and explore the best approach to
support our system.
Fanout on write. With this approach, news feed is pre-computed during write time. A new
post is delivered to friends’ cache immediately after it is published.
Pros:
• The news feed is generated in real-time and can be pushed to friends immediately.
• Fetching news feed is fast because the news feed is pre-computed during write time.
Cons:
• If a user has many friends, fetching the friend list and generating news feeds for all of
them are slow and time consuming. It is called hotkey problem.
• For inactive users or those rarely log in, pre-computing news feeds waste computing
resources.
Fanout on read. The news feed is generated during read time. This is an on-demand model.
Recent posts are pulled when a user loads her home page.
Pros:
• For inactive users or those who rarely log in, fanout on read works better because it will
not waste computing resources on them.
• Data is not pushed to friends so there is no hotkey problem.
Cons:
• Fetching the news feed is slow as the news feed is not pre-computed.
We adopt a hybrid approach to get benefits of both approaches and avoid pitfalls in them.
Since fetching the news feed fast is crucial, we use a push model for the majority of users.
For celebrities or users who have many friends/followers, we let followers pull news content
on-demand to avoid system overload. Consistent hashing is a useful technique to mitigate the
hotkey problem as it helps to distribute requests/data more evenly.
Let us take a close look at the fanout service as shown in Figure 11-5


• Notification service: inform friends that new content is available and send out push
notifications.

Newsfeed building

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/dOvY7_dVnE58HKIBQ6nJl.png?ixlib=js-3.7.0 "image.png")

• User: a user sends a request to retrieve her news feed. The request looks like this:
/ v1/me/feed.
• Load balancer: load balancer redirects traffic to web servers.
• Web servers: web servers route requests to newsfeed service.
• Newsfeed service: news feed service fetches news feed from the cache.
• Newsfeed cache: store news feed IDs needed to render the news feed.



Newsfeed retrieval deep dive

1. A user sends a request to retrieve her news feed. The request looks like this: /v1/me/feed
2. The load balancer redistributes requests to web servers.
3. Web servers call the news feed service to fetch news feeds.
4. News feed service gets a list post IDs from the news feed cache.
5. A user’s news feed is more than just a list of feed IDs. It contains username, profile
picture, post content, post image, etc. Thus, the news feed service fetches the complete
user and post objects from caches (user cache and post cache) to construct the fully
hydrated news feed.
6. The fully hydrated news feed is returned in JSON format back to the client for
rendering
Cache architecture

Cache is extremely important for a news feed system. We divide the cache tier into 5 layers as shown

![image.png](https://eraser.imgix.net/workspaces/sxabTBCPWukdhhQ5jfZK/pqzq4S07fqcma47xGOPbSlv9Jtt1/O_Clpta8tlwwmXCePUYHs.png?ixlib=js-3.7.0 "image.png")

News Feed: It stores IDs of news feeds.
• Content: It stores every post data. Popular content is stored in hot cache.
• Social Graph: It stores user relationship data.
• Action: It stores info about whether a user liked a post, replied a post, or took other
actions on a post.
• Counters: It stores counters for like, reply, follower, following, etc.

