---
layout: post
title: "Testing Google Cloud Platform PubSub with Testcontainers"
date: 2020-08-24 09:00
comments: true
categories: [Kotlin, Testcontainers, GCP]
---

I wanted to be able to write tests against our Google Cloud Platforms PubSub implementation. Unfortunately, the way the SDK is written, it doesn't exactly lend itself to easy testing. It's incredibly verbose for something that is just sending a bunch of http requests!

For integration testing I considered a couple of options:

### [Wiremock](http://wiremock.org/docs/stubbing/)

You could mock out the google cloud platform api. You can then write a bunch of assertions against what's been called and stub out responses. The problem is that google is rather bad at supporting their apis; they have a tendency just to deprecate thing without much warning. Consequently, there's a higher degree of maintenance required in keeping your tests up to date. 

### [GCloud Emulator](https://cloud.google.com/spanner/docs/emulator)

Google provides a local, in-memory emulator, which you can use to develop against. In theory, they should keep it up to date, but who knows! It's also currently in beta, which means google might decide to drop it without any notice.

## GCloud api and the java sdk

A couple of gotchas with the SDK. They could be rectified with a better implementation.

You would think that creating a topic would be easy enough:

We can call the `Topic createTopic(String name)` method like this:

`createTopic("topicName")` 

It semes like a logical thing to do. Except it doesn't work. You get a runtime exception.

`INVALID_ARGUMENT: Invalid [topics] name: (name=topicName)`

What's annoying is that the name isn't what you want to call it, it's what gcp pubsub wants to call it and it's not a name it's a uri! So the name is a fully qualified uri like this `projects/projectId/topics/topicName`. It is documented but it's a poor implementation in my opinion.

There's a builder that you can use `TopicName.of(projectId, topicName)` and everything works fine. 

The other frustrating thing is that you need to pass the projectId everywhere, there's no way to set it by default on the client like you can in the gcs sdk?!

So now that you've created your topic you want to list them so reach for `listTopics`. Again there's an option to pass a string; if you get the format wrong again, the sdk will just spin forever! Rather than giving you a 404?! 

Once you get past all this friction, you might be able to write some code!

## Starting the emulator with TestContainers

```kt

object PubSubTestContainer {

    private val logger = KotlinLogging.logger {}

    val pubsubEmulator by lazy { startEmulator() }

    private fun startEmulator(): KGenericContainer {
        return KGenericContainer("gcr.io/google.com/cloudsdktool/cloud-sdk")
            .withExposedPorts(8085)
            .withCreateContainerCmdModifier {
                it.withEntrypoint("gcloud", "beta", "emulators", "pubsub", "start", "--host-port=0.0.0.0:8085")
            }
            .waitingFor(Wait.forLogMessage(".*Server started.*", 1))
            .apply {
                start()
                followOutput {
                    logger.debug { it }
                }
            }

    }

}

```

Now that we've got the emulator running, we can use it in the various client implementations that the sdk provides. Unfortunately, there's not a single point of entry for using the PubSub sdk. It forces you to create a client for each subsystem.

## Creating a topic

To create a topic you need to use the `TopicAdminClient`. When using the emulator you need to set two properties; the transport channel and the credentials to use.

```kt
val channel = ManagedChannelBuilder
        .forAddress(pubsubEmulator.host, pubsubEmulator.firstMappedPort)
        .usePlaintext()
        .build()

val channelProvider = FixedTransportChannelProvider.create(
    GrpcTransportChannel.create(channel))

val topicAdminClient =TopicAdminClient
    .create(
        TopicAdminSettings.newBuilder()
            .setTransportChannelProvider(channelProvider)
            .setCredentialsProvider(NoCredentialsProvider.create())
            .build())

```

We can then start to write some test around the creation of topics.

```kt
@Test
fun `can create and list topics`() {
    topicAdminClient.createTopic(TopicName.of("project-id", "a-topic"))

    val topic = topicAdminClient
        .listTopics(ProjectName.of("project-id"))
        .iterateAll()
        .first()

    assertThat(topic.name, equalTo("projects/project-id/topics/a-topic"))
}
```

Great we have a topic, now we want to publish some messages to that topic. 

## Publishing

We want to create a message that we publish, you can't just publish a string. That would, of course, be too easy and to intuitive. No, first you need to create a `ByteString` then create a PubSubMessage.

```kt
val data = ByteString.copyFromUtf8("payload")
val pubsubMessage = PubsubMessage.newBuilder()
        .setData(data)
        .build()
```
Seems like a lot of code for not very much!

We've already seen how to create a topic previously.

`val topic = topicAdminClient.createTopic(TopicName.of("project-id", "another-topic"))
`

Can we call `topic.publish` or something like that? Not a chance. We need to create a `Publisher` and give it a topic to publish to. So we end up with:  

`Publisher.newBuilder(topic).build().publish(pubsubMessage)` except that doesn't compile because the `newBuilder` doesn't take a topic it takes a topic name. Okay, is topic.name of type `TopicName` of course not, it's a string.

`Publisher.newBuilder(topic.name).build().publish(pubsubMessage)`


Does it work, hell no! The problem is that it's trying to connect to gcp rather than our locally running emulator. This makes sense as the sdk needs to send something over the wire somewhere! It's just it doesn't follow the convention of calling something a client. `PublisherClient` would make more sense or better yet `TopicPublisherClient` as each publisher is built for a single topic.

```kt
val publisher = Publisher
            .newBuilder(topic.name)
            .setCredentialsProvider(NoCredentialsProvider.create())
            .setChannelProvider(channelProvider)
            .build()

publisher.publish(pubsubMessage)
```

Let's go ahead and subscribe to a topic. First, you need to create a subscription then subscribe to events on that subscription. The idea is that you can have multiple subscriptions to one topic and then you collect messages from that subscription. 

*The subscription needs to exist before you publish the message or it will be lost*

The signature for `createSubscription` is fairly stupid. We're encouraged to use `ProjectSubscriptionName.of` which means that we have to use a `TopicName` for the second argument. Can you get a `TopicName` from a `Topic`? No, so we end up with this nasty looking mess:

```kt
val subscription = subscriptionAdminClient.createSubscription(
        ProjectSubscriptionName.of("project-id", "subscription"),
        TopicName.parse(topic.name),
        PushConfig.getDefaultInstance(),
        30
)
```

We now need to pull message for that subscription. We can do this synchronously (there are methods for doing this async but I prefer to keep it simple).

```kt

val subscriberStubSettings = SubscriberStubSettings.newBuilder()
        .setCredentialsProvider(NoCredentialsProvider.create())
        .setTransportChannelProvider(channelProvider)
        .build()


val pullRequest = PullRequest.newBuilder()
            .setMaxMessages(1)
            .setSubscription(subscription.name)
            .build()

val subscriber = GrpcSubscriberStub.create(subscriberStubSettings)

val pullResponse = subscriber.pullCallable().call(pullRequest)
```
We then need to ack the message once we've done what we need to do.

```kt
val acknowledgeRequest = AcknowledgeRequest.newBuilder()
            .setSubscription(subscription.name)
            .addAckIds(pullResponse.getReceivedMessagesList().first().ackId)
            .build()

subscriber.acknowledgeCallable().call(acknowledgeRequest)

```

Fairly straight forward.


## Summary

Using GCP PubSub is not that simple to get started with and is not that easy to test.

All the code is available over at [github](https://github.com/antonydenyer/test-google-cloud-pubsub)