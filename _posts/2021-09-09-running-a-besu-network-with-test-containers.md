---
layout: post
title: "Running a besu network with test containers"
date: 2021-09-09 09:00
comments: true
categories: [Programming, Besu, Ethereum]
---

[Testcontainers](https://www.testcontainers.org/) is a great project that helps you orchestrate throwaway docker container instances. I wanted to integrate test some smart contracts that I'd been working on with a real ethereum network rather than with [ganache](https://www.trufflesuite.com/docs/ganache/overview).

## Quorum Dev Quickstart

[ConsenSys/quorum-dev-quickstart](https://github.com/ConsenSys/quorum-dev-quickstart) provides a great point of reference for getting started with Besu. The problem is it's focused on having a long-ish running network on your machine rather than a throwaway network for unit tests. I used the docker-compose that was generated from the quick start as inspiration. 

## Testcontainers

Initially, I wanted to have a single docker image and use test containers to orchestrate multiple nodes. This proved to be a real pain, so I switched back to using a docker-compose.

Here's what I came up with:

```kotlin

class BesuNetwork {
    private val logger = LoggerFactory.getLogger(BesuNetwork::class.java)

    private val nodes: Map<String, BesuNode>

    private val instance: KDockerComposeContainer by lazy { KDockerComposeContainer(File("src/test/resources/docker-compose.yml")) }

    class KDockerComposeContainer(file: File) : DockerComposeContainer<KDockerComposeContainer>(file)

    init {
        instance.withEnv("BESU_VERSION", System.getenv("BESU_VERSION"))

        val serviceNames = listOf("miner", "alice", "bob")

            serviceNames
            .forEach {
                val serviceName = "${it}_1"
                instance
                    .withExposedService(serviceName, 8545, Wait.forListeningPort())
                    .waitingFor(serviceName, Wait.forLogMessage(".*Ethereum main loop is up.*", 1))
                    .withLogConsumer(
                        serviceName,
                        Slf4jLogConsumer(logger).withPrefix(it)
                    )
            }

        instance.start()

        nodes = serviceNames
            .associateWith {
                String serviceName = node.getName() + "_1";
                  val rpcUrl =
                    "http://${instance.getServiceHost("${it}_1", 8545)}:${instance.getServicePort("${it}_1", 8545)}"
                val web3j = JsonRpc2_0Web3j(HttpService(rpcUrl), 2000, Async.defaultExecutorService())
                val credentials =
                    Credentials.create(getResource(BesuNetwork::class.java, "/config/$it/key").readText())

                BesuNode(
                    web3j,
                    credentials
                )
            }
    }

    fun stop() = instance.stop()
    fun get(key: String) = nodes[key]!!
}

data class BesuNode(val web3j: JsonRpc2_0Web3j, val credentials: Credentials)
```

When you create the class we are waiting for all nodes to start and then we're providing some convenience methods to communicate to the node.

The test code looks reasonble too:

```kotlin

class PublicContractTest {

    companion object {

        val instance = BesuNetwork()

        @AfterAll
        @JvmStatic
        internal fun afterAll() {
            instance.stop()
        }
    }

    @Test
    fun canDeployPublicContract() {
        val alice = instance.get("alice")

        val simpleStorageContract = SimpleStorage.deploy(
            alice.web3j,
            alice.credentials,
            DefaultGasProvider()
        ).send()

        simpleStorageContract.set(BigInteger.valueOf(42)).send()

        assertThat(simpleStorageContract.get().send()).isEqualTo(BigInteger.valueOf(42))
    }
}
```

Full code can be found at [https://github.com/antonydenyer/besu-testcontainers](https://github.com/antonydenyer/besu-testcontainers)