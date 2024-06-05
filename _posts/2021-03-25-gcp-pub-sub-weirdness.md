---
layout: post
title: "Be warned when using GCP Pub Sub SDK Libraries."
date: 2021-03-05 09:00
comments: true
categories: [Programming, GCP, PubSub]
---
GCP PubSub is an off the shelf hosted message queue system. It offers some reasonably standard features. Conceptually GCP supports topics and subscriptions. You post a message onto a topic, and they fan out to subscriptions. Each subscription can have one or many consumers, with messages being load balanced between them.

# Initial trade-offs with GCP
A fairly common practice is to take a message from the queue, do some work then acknowledge the message once the work is complete. This means that if there any problems processing the message it will get put back on the queue. To do this, you usually stick some imperative code around the entry point of your consumer.

```
try {
    doWork(msg.body)
    msg.ack()
} catch(allTheThings) {
    msg.nack()
}
```
The problem from the queue system point of view is what should you do when a message is neither ack'd or nack'd. Well, you usually have a timeout of some description. When the timeout is hit, the message goes back on the queue. In the case of GCP, it's [Acknowledgement deadline](https://cloud.google.com/pubsub/docs/admin), and it has a maximum of 10 minutes.

# The Problem
The scenario we had was that we had about 60 messages that would each take about 10-15 minutes. No big deal, we thought, the message will go back on the queue, and we can check to see if we've already run it.


```
try {
    if (taskDoneExists(msg.body.correlationId)) {
        doWork(msg.body)
        storeTaskDone(msg.body.correlationId)
    }
    msg.ack()
} catch(allTheThings) {
    msg.nack()
}
```

Worst case, we end up running the same task a few times on different consumers; this would happen if we had multiple consumers but given we only had a single consumer shouldn't be a problem.

# The Error
What happened is that everything ran as expected for 60 minutes. Then we would see a bunch of messages ending up on the dead letter queue with a deadline exceed error. What was confusing was that we hadn't taken the message from the queue to start working on it yet?!

# The 'helpful' SDK

All the default docs for the various SDKs tell you that it's straightforward to receive messages from a subscription. It takes you through some code you need to create a callback for an asynchronous process to run on a background thread.  

https://cloud.google.com/pubsub/docs/samples/pubsub-quickstart-subscriber

The first thing that you need to be aware of that is hidden way down in the SDK is that by default, it will create four threads to consumer messages or a thread per CPU core. In production, we have a 32 core machine, so we had 32 consumers, which caused us some confusion. But at least we could see the messages going through on different threads. To make things more straightforward for us, we reduced the executor thread count to 1.

```
val executorProvider = InstantiatingExecutorProvider.newBuilder().setExecutorThreadCount(1).build()
```
Then pass it in when you create the SubscriptionClient

The problem was we were still seeing the same behaviour where messages were ending up on the dead letter queue after an hour.

It turned out that what was happening is that the SDK will prefetch a bunch of message for you! And then under the hood, if you haven't handled them, it would (https://cloud.google.com/pubsub/docs/reference/rpc/google.pubsub.v1#google.pubsub.v1.ModifyAckDeadlineRequest)[Modify the Ack Deadline] on your behalf. Except it would only extend it for 1 hour and ultimately decided not to extend it any longer. So what was happening was the SDK was taking a message from the queue that we weren't ready for, holding it in memory and telling the queue that we're still looking into it. 

So to get around this problem, we had to `setMaxAckExtensionPeriod` to a more desirable value.


# Summary

The frustration I had was that nowhere in the docs is there any mention of prefetch, and all of the docs associated with the acknowledgement deadline don't mention it. Worst still, you can't easily specify the number of messages you wish to prefetch; it's a combination of CPU cores and thread count that defines this. The other problem with GCP PubSub is that you can't see how many messages are in-flight; you can only see the number of un-ack'd messages. If we had been able to see the in-flight count, it would have been evident that we had taken more messages than we wanted to from the queue.