---
layout: post
title: "Testing Google Cloud Storage with Testcontainers and MockK"
date: 2020-08-14 17:52
comments: true
categories: [Kotlin, Testcontainers, MockK, Hamkrest, GCS]
---

I wanted to be able to test our integration with Google Cloud Storage. We've got a simple private bucket with some text blobs. The code to get the data out is straightforward.

``` kt
val storage: Storage = StorageOptions.newBuilder()
    .build()
    .service

fun get(file: String): String {
    val buffer = ByteArrayOutputStream()

    storage.get("my_bucket", file)
            .downloadTo(buffer)

    return buffer.toString()
}
```

# Testing with MockK

But to test this code is a bit of a pig. The sdk implementation doesn't have a simple way to get the file because you have to pass a `MemoryStream` to `downloadTo` as an argument; it then gets filled up with data. 

To be able to Unit test the code we need to be able to capture the input argument in `downloadTo` and fill the argument with stubbed data. Thankfully with MockK it's reasonably easy using [a CapturingSlot](https://mockk.io/#capturing).

```kt
@Test
fun `can get file as string`() {
    val blob = mockk<Blob>()
    val stream = slot<OutputStream>()

    every {
        blob.downloadTo(capture(stream))
    } answers {
        stream.captured.write("content".toByteArray())
    }

    val storage: Storage = mockk()
    every { storage.get("my_bucket", "my_file.txt") } returns blob

    val bucket = TextBucketService("my_bucket", storage)

    val result = bucket.get("my_file.txt")

    assertThat(result, equalTo("content"))
}
```

It's fairly gnarly code for what is a fairly simple function. The first thing is that `storage.get` returns a Blob, now you can't just new up a Blob, that would be way to easy. You need to use a Builder, to construct a Blob rather than a constructor. The problem is `Blob.newBuilder` returns `BlobInfo` rather than a `Blob`. So instead we end up creating a `mockk<Blob>()` so that we can return it when someone calls get. The other thing I don't like about this code is that it reads out of order. The first thing we see is the mock about the stream, then we see a mock about the bucket. But the code gets the bucket then the stream. The test code is backwards compared to the implementation code. It's the sort of thing that just bugs me; it feels like some sort of cognitive dissonance where I both get the file first and last. 

If the library had been TDD'd I suspect it would be a lot easier to work with.

# Integration Test with Testcontainers

First, we need a stub/fake/mock/canned data server for GCS. Luckily someone has already done this [fake-gcs-server](https://github.com/fsouza/fake-gcs-server/).

Then we need to integrate it with [Testcontainers](https://www.testcontainers.org/).


```kt
@Container
private val gcs = KGenericContainer("fsouza/fake-gcs-server")
    .withExposedPorts(4443)
    .withClasspathResourceMapping("data", "/data", BindMode.READ_WRITE)
    .withCreateContainerCmdModifier {
        it.withEntrypoint("/bin/fake-gcs-server", "-data", "/data", "-scheme", "http")
    }
```

We can then use this with our storage service

```kt
val storage: Storage = StorageOptions.newBuilder()
    .setHost("http://${gcs.host}:${gcs.firstMappedPort}")
    .build()
    .service
```

Luckily the sdk allows us to specify the host!

Then in our project, we just need to provide some canned data in the resources folder, and our test becomes much more readable.

```kt
val bucket = TextBucketService("my_bucket", storage)

val result = bucket.get("my_file.txt")

assertThat(result, equalTo("content"))
```

# Conclusion

I don't like the unit test in this case. I don't think it adds much value. I feel like the main thing is to keep the integration point simple. Then mock that service in your classes that use it. The other thing is that if you want a true integration test, you should be using a test instance of the actual thing. This is relatively easy with things that are open source, but when you're using a proprietary, non-standard solution, you have to rely on emulators. You could hit the actual service, but this can just lead to other problems.

A full code example available over on [github](https://github.com/antonydenyer/test-google-cloud-storage)