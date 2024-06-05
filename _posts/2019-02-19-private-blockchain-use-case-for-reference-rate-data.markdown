---
layout: post
title: "Private Blockchain Use Case For Reference Rate Data"
date: 2019-02-19 14:24:09 +0000
comments: true
redirect_from: '/blog/2019/02/19/private-blockchain-use-case-for-reference-rate-data/'
categories: ["blockchain", "quorum", "smart contracts", "LIBOR", "ethereum"]
---

## Reference Rate Data Use Case

Many financial instruments use reference data to perform their calculations. For instance, LIBOR is widely used as a baseline for contracts. Consequently, there is a need to have a trusted source of truth about that data for a given point in time. Currently, that service is proved by Thomson Reuters.

## TL;DR
You could use blockchain to make data access cheaper.

### LIBOR Panel
First a bit of background about how it gets calculated. The panel consists of several large banks who all contribute to the calculation of rates. In essence, each bank is asked at what rate they would be willing to lend to the other banks. This is then packaged together with the average being calculated and published every day.

### Greater transparency
By recording the panel members responses on a blockchain, you can improve the level of trust people have in the panel. Each panel member would want to keep their rates private. While the averaged calculated rate would be made public. Each panel member has an interest in maintaining the validity of the network as the panel is already a consortium of members it easy to represent using a Quorum network. With each member running a private permissioned node and enclave.

### Walled Garden
The problem with running a private network is that people don't have easy access to the data. What you want is the calculated rate to be available on a public network. While ensuring that the trust and validation of the original contract is maintained.

## Trustless cross-chain interaction
To get the data onto a public network in a trustless way is hard. There has to be some interaction at the application level. Someone needs to take an action to publish the data. However, there are some things you can do to improve verification beyond just putting another hash on the blockchain (notarize). You could use something like [Ion Interoperability Framework ](https://github.com/clearmatics/ion) to validate that the data was verified by someone on the reference data network. The problem is that you can not guarantee that that is always the truth. At best you can attest that the data was correct at that point in time. What you need is a further application level confirmation. You need to have every panel member also confirm that the public chain record is the same as the private chain record that they have. 

## Summary
You have a situation with multiple parties that have a vested interest in the integrity of the published data. The cost of accessing the data would be reduced as it would be available on a public network. Furthermore, stronger guarantees can be provided about the truthfulness of the data than can be achieved at present with a single information provider.  