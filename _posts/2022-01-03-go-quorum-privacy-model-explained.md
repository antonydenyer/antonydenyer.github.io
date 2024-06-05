---
layout: post
title: "GoQuorum Privacy model explained"
date: 2022-01-03 09:00
comments: true
categories: [Enterprise Ethereum, GoQ, Ethereum]
---

In GoQuorum, members of a network can communicate privately using smart contracts. The way it works is relatively simple. First, a standard ethereum transaction occurs in public with the input field containing an enclave key. Then, that enclave key is used to fetch the private contract from your private enclave. 

To deploy a private contract onto a GoQuorum network, you must first serialize an ethereum smart contract. Then you must distribute that smart contract to the members you want to see it. GoQuorum uses tessera, a private p2p enclave network, to achieve this. Finally, a transaction is submitted to the blockchain with the enclave key as the input.


Let's run through an example to explore how that looks to different parties. Say we've deployed a private contract onto a GoQuorum network. 

All participants will have the same result for `eth_getTransaction`.

`eth.getTransaction("0x210bfb0bb0cc390a4285297eedadc89746caf47dcca8d224f3aaa40000545771")`

```js
{
  blockHash: "0x9cd2af7571a35f86685c15cf83dc04215bb84c5e967e1d08b34ed114913cf95f",
  blockNumber: 7,
  from: "0xf0e2db6c8dc6c681bb5d6ad121a107f300e9b2b5",
  gas: 150050,
  gasPrice: 0,
  hash: "0x210bfb0bb0cc390a4285297eedadc89746caf47dcca8d224f3aaa40000545771",
  input: "0x1dd623a1b8e00b978db147519fdc8f9440325569f748d79a71aa69bfc3b7b5eafbf122d016cb71e9740fede7fe4460e37a2fb9b8d5d82877134c7c10a57b3790",
  nonce: 0,
  r: "0x3a4d8d0f0da2529f83c50a57739b6a071d3210ec06028a6c9c464e1226eee3cd",
  s: "0x3fe4cc5b20aa4156bec3a06df79a200347ff345cc58df004c2ee7fd96e4a8bfd",
  to: null,
  transactionIndex: 0,
  type: "0x0",
  v: "0x26",
  value: 0
}
```

Notice that the input is `0x1dd623a1b8e00b978db147519fdc8f9440325569f748d79a71aa69bfc3b7b5eafbf122d016cb71e9740fede7fe4460e37a2fb9b8d5d82877134c7c10a57b3790` this decodes to `HdYjobjgC5eNsUdRn9yPlEAyVWn3SNeacappv8O3ter78SLQFstx6XQP7ef+RGDjei+5uNXYKHcTTHwQpXs3kA==` which is used to lookup the private contract in your enclave.

Because the privacy model being used is `restricted`, only the members who should have access to the contract will have it in their enclave.

For the nodes that have the data in their enclave, you can call `eth_getQuorumPayload`:

`eth.getQuorumPayload("0x1dd623a1b8e00b978db147519fdc8f9440325569f748d79a71aa69bfc3b7b5eafbf122d016cb71e9740fede7fe4460e37a2fb9b8d5d82877134c7c10a57b3790")`

```js
"0x608060405234801561001057600080fd5b506040516102043803806102048339818101604052602081101561003357600080fd5b81019080805190602001909291905050507fc9db20adedc6cf2b5d25252b101ab03e124902a73fcb12b753f3d1aaa2d8f9f53382604051808373ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019250505060405180910390a18060008190555050610154806100b06000396000f3fe608060405234801561001057600080fd5b50600436106100415760003560e01c80632a1afcd91461004657806360fe47b1146100645780636d4ce63c14610092575b600080fd5b61004e6100b0565b6040518082815260200191505060405180910390f35b6100906004803603602081101561007a57600080fd5b81019080803590602001909291905050506100b6565b005b61009a610115565b6040518082815260200191505060405180910390f35b60005481565b7fc9db20adedc6cf2b5d25252b101ab03e124902a73fcb12b753f3d1aaa2d8f9f53382604051808373ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019250505060405180910390a18060008190555050565b6000805490509056fea264697066735822122027c2c4639230624c056430fb1d781389b791d56f4d6fe2ca494a46a171f5433164736f6c63430007060033000000000000000000000000000000000000000000000000000000000000002f"
```

This returns the private contract data. At the end of the data, you can see the constructor is called with `2F` ie `47`.

When GoQuorum sees a private transaction, it fetches the payload from its enclave then applies the contract (if present) to the private state. 

Note, in GoQuorum; there are only two states available public and private.

# Observations

Note that everybody can get the transaction receipt for the transaction, but they will have different results. For non-members you will get:

```js
{
  blockHash: "0x9cd2af7571a35f86685c15cf83dc04215bb84c5e967e1d08b34ed114913cf95f",
  blockNumber: 7,
  contractAddress: "0x00ffd3548725459255f1e78a61a07f1539db0271",
  cumulativeGasUsed: 0,
  from: "0xf0e2db6c8dc6c681bb5d6ad121a107f300e9b2b5",
  gasUsed: 0,
  isPrivacyMarkerTransaction: false,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x210bfb0bb0cc390a4285297eedadc89746caf47dcca8d224f3aaa40000545771",
  transactionIndex: 0,
  type: "0x0"
}
```

and for members, you will get 

```js
{
  blockHash: "0x9cd2af7571a35f86685c15cf83dc04215bb84c5e967e1d08b34ed114913cf95f",
  blockNumber: 7,
  contractAddress: "0x00ffd3548725459255f1e78a61a07f1539db0271",
  cumulativeGasUsed: 0,
  from: "0xf0e2db6c8dc6c681bb5d6ad121a107f300e9b2b5",
  gasUsed: 0,
  isPrivacyMarkerTransaction: false,
  logs: [{
      address: "0x00ffd3548725459255f1e78a61a07f1539db0271",
      blockHash: "0x9cd2af7571a35f86685c15cf83dc04215bb84c5e967e1d08b34ed114913cf95f",
      blockNumber: 7,
      data: "0x000000000000000000000000f0e2db6c8dc6c681bb5d6ad121a107f300e9b2b5000000000000000000000000000000000000000000000000000000000000002f",
      logIndex: 0,
      removed: false,
      topics: ["0xc9db20adedc6cf2b5d25252b101ab03e124902a73fcb12b753f3d1aaa2d8f9f5"],
      transactionHash: "0x210bfb0bb0cc390a4285297eedadc89746caf47dcca8d224f3aaa40000545771",
      transactionIndex: 0
  }],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000002001000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x210bfb0bb0cc390a4285297eedadc89746caf47dcca8d224f3aaa40000545771",
  transactionIndex: 0,
  type: "0x0"
}

```

Notice, the only difference is the logs, that is, the logs emmited from the private contract. Everyone knows that you `0xf0e2db6c8dc6c681bb5d6ad121a107f300e9b2b5` have created the contract `0x00ffd3548725459255f1e78a61a07f1539db0271`.


# Summary

The main thing to think about with GoQuorum is that it supports private smart contracts and not private transactions.