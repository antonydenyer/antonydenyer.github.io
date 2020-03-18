---
layout: post
title: "Why peer count matters for distributed systems"
date: 2020-02-26 08:00:00 +0000
comments: true
categories: ["ethereum", "blockchain", "consensus"]
---

TL;DR

15 nodes are probably fine

# Overview

Ethereum is a decentralised application platform; it is a distributed unstructured peer to peer system built up of many nodes. Because it is unstructured, it means that every node must have a complete copy of all the data on the network. It also means that a node will act as both server (to other peers) and client (fetches data from peers) to the network. Because each node must fetch all the data for that blockchain, it means that it's open to attack from malicious actors who might want to propagate corrupted data to you. To mitigate this problem Satoshi Nakamoto [in the bitcoin white paper](https://bitcoin.org/bitcoin.pdf) proposed that this could be solved by introducing a 'proof of work' consensus algorithm.


# What has this got to do with Peer Count?

Simply put, you need to have peers to be able to have a network. Otherwise, you're just talking to yourself! You could run a node with a single peer that's connected to the broader network. The downside is that you must trust, completely, that single peer. At the other extreme you could try and connect to all available peers, the problem is that it doesn't scale. You can not talk to everyone in a timely manner. What you need to have is a reasonable number of peers whereby the chances of more than half of them being malicious is low.

# How do you prevent yourself from being hijacked 

The way you discover peers needs to be random. The problem is you need to start somewhere, with ethereum mainnet this is achieved via bootnodes. They are maintained by the ethereum foundation and act as a lookup for other nodes. Once you have some peers you can ask for some more peers (neighbours). Your client can walk the network to find sufficiently random peers to connect to. 

# Check your peers
It's a good idea to make sure that the peers that you've connected to are sufficiently distributed. There's no point connecting to 15 peers that are owned by the same actor. There's no real fool proof way of doing this, but you can make a start by checking `admin.peers` from a geth console or by using [epirus](https://docs.epirus.io/features/#peers)



References:

https://github.com/ethereum/devp2p/blob/master/rlpx.md 

https://github.com/ethereum/devp2p/wiki/Discovery-Overview 

https://medium.com/loom-network/understanding-blockchain-fundamentals-part-1-byzantine-fault-tolerance-245f46fe8419 
