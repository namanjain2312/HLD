# NotifactionSystem

Three types of notification formats : 

1. Mobile Push Notification 
2. SMS Message 
3. Emails


Defining Scope

Question 1. What type of notification does the system support ? 

Answer : Push Notification , SMS Message , Email 

Question 2. Is it real time ?

Answer : Let say it is a soft real time , meaning we want to receive our notification as soon as possible , but a slight delay could be accepted at high payload .

Question 3. What are the supported devices ?

Answer : IOS Devices , Android Devices laptop/desktop 

Question 4. What triggers notification ?

Answer : Notification could be triggered by client application . They could also be scheduled on the server side .

Question 5. Will users be able to opt - out?

Answer : Yes , user who opt out will not receive any further more notification .

Question 6. How many notification are send out each day ?

Answer : 10 million mobile push notification , 1 million SMS messages and 5 million emails .



Notification 

1. Apple push notification
    1.  Provider : Service which builds and send notification 
    2. APNs : Apple Push Notification service (Remote service provided by apple to propagate push notification to IOS devices ) It takes two inputs 
        1. Device Token
        2. Payload
    3. IOS Device : End client which receives push notification
2. Android Push Notification 
    1. Provider
    2. FCM : Firebase Cloud Messaging (FCM)
    3. Android Device
3. SMS
    1. Provider
    2. SMS third party services like Twilio , Nexmo 
    3. SMS
4. Email 
    1. Provider
    2. Email Service like Send Grid , MailChimp
Contact Info Gathering 

1. To send notification we need to gather mobile device tokens , phone number , email id
2. Gathered when user install app or sign in the app 
Notification Logs 

1. Role of them is to provide data reliability and data loss
Notification Duplication

1. To avoid this we use dedupe mechanism like when for the first time the notification arrives we could check weather it is seen before by checking its event id , if it is seen before it is discarded otherwise it is send . (This happen over worker level )
Notification Template 

1. Usually large amount of notification is been send each day , we could use some message templates to avoid building every notification from scratch , maintain consistent format of notification , reduce the margin error and saving time.
Notification Setting 

1. User generally set the notification preferences on there end , thus before sending any notification to user we first check if user is opted for the same or not .
Rate Limiting 

1. To avoid an user to receive a lot of notification at the same time ,  a rate limiter is been set on a user , so this avoid the user to turn off notification permanently for that app 
2. Also rate limiter could be send at all those third party services which pushes notification , which usually provides a limit / sec to send the request , thus if we cross those limit these third party notification sending services will block us for a few time before we could send any other notification request .
Retry Mechanism 

1. When a third party app fails to send notification , the notification will be added to the message queues for retrying , if the problem till persists , an alert would be send out for the developers for the same 
Authentication 

1. For IOS and Android , app key and app secret are used to send the notification , only authenticated or verified client are allowed to send the push notification using the api .
Monitor Queued Notification 

1. If the queued notification size is large , it mean our workers are not pushing the notification as fast as they need to be , so add new workers at that point of time .
Event Tracking 

1. Metrics like open rate , click rate and engagement are important to understand the customer behavior and should be tracked 
Conclusion

1. Types of notification 
2. Gather contact info of user






