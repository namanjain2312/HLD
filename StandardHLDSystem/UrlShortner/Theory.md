# Url Shortner

Requirement Analysis 
1>How short our url should be ?
	As much as you can 
2>Traffic 
	10 million url / day  
	=>3650 M / year 
	=>If we need to store data fro 100 year => 365000 M => 365 Billion url 
	and now say our api should be supported for 100 year
3>What all character i can use 
	0-9 , / a-z / A-Z = 62 character
	=>62^6 = 56 Billion and 62^7 = 3.5 Trillion 
	=>We need 7 characters 
4>Now we could not use one db as it will be SPOF, scalableinty issue , if we have multiple db servers then we need to put them in sync in order to provide the uniqueness and avoid duplicate keys and synchronization .

4>How to generate these hash values 
	a>Use hash function
MD5 - it is a 128 bit hash function , it mean it wil generate 128/8 = 16 bytes number , in 1 byte = 8 bit , two hexadecimal number can come , so in 16 bytes it will generate 32 length hexadecimal number , but we need only 7 so do not use it.
SHA-1 - 160 bit ans with same calculation it will generate 40 length .
	Now if we select first 7 digit of the above generated number , there would be a lot of duplicate and collision due to obvious reason k, as it generate 32 digit unique number but if we trim and only first 7 then duplicancy would be there .

	b>Use Base 62 encoding
		Problems 
			BAse 62 needs an id generator .
			Length could be diff
		Solution
1>Ticket Sever 
It uses concept of centralized auto incremanent value , but if fails whole service fails 
2>Snowflake 
TimeStamp | MachineId | Sequence Number
This could be used 
Easy to implement .
3>Zookeper - Distribute application , they can coordinate with each other reliable , and perform some operation
So we have total 62^7 number s, so zzookeeper would create a range, 
Divide the the total number in range like 0-1M,1-2M,2-3M like that and then each range , now whenever the request come for a partivualr worker thread then it does to the particular worker thread and then increment the previous number , remember 

Sp divide 3.5 Trillion in 1 1 M range and then get an number id , now base it with 62 then do padding till 7 didigit if required .

So in this way we are guaranteed that we would generate unique id in the distributed envrinoment.




![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/4cssVIWRkpTVzBIgLmi1q.png?ixlib=js-3.7.0 "image.png")





Problems with above design :

There is only one WebServer which is single point of failure (SPOF)
System is not scalable
There is only single database which might not be sufficient for 60 TB of storage and high load of 8000/s read requests
To cater above limitations we :

Added a load balancer in front of WebServers
Sharded the database to handle huge object data
Added cache system to reduce load on the database.
We will delve further into each component when we will go through the algorithms in later sections






![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/MEiskjZ5nw0fJPxUP33Z-.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/A-b4da5hI-IBJSXfN9DJl.png?ixlib=js-3.7.0 "image.png")

![image.png](https://eraser.imgix.net/workspaces/i6q9GN2aZmfBzzDiboTF/pqzq4S07fqcma47xGOPbSlv9Jtt1/_yWTbSCkZ-2ihrwTcprTX.png?ixlib=js-3.7.0 "image.png")







References

3>https://medium.com/@sandeep4.verma/system-design-scalable-url-shortener-service-like-tinyurl-106f30f23a82

4>https://www.youtube.com/watch?v=C7_--hAhiaM&list=PL6W8uoQQ2c63W58rpNFDwdrBnq5G3EfT7&index=8&ab_channel=Concept%26%26Coding-byShrayansh
5>https://leetcode.com/discuss/interview-question/system-design/124658/Design-URL-Shortening-service-like-TinyURL


