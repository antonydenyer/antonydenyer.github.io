---
layout: post
title: "Besu Privacy model explained"
date: 2022-03-03 09:00
comments: true
categories: [Enterprise Ethereum, Besu, Ethereum, hyperledger besu]
---

In hyperledger besu, members of a network can communicate privately using private ethereum transactions. The way it works is relatively simple. First, a standard ethereum transaction occurs publicly with an input field containing an enclave key. Then, that enclave key is used to fetch the private transaction. A private transaction is the same as a public ethereum transaction with additional privacy metadata. The privacy metadata contains the `privateFrom` and the `privateFor` or the `privacyGroupId`.

Besu will then apply that transaction to a privacy group; the `privacyGroupId` is either specified explicitly in the transaction or deterministically derived by combining `privateFrom` and `privateFor`. The privacy group is a complete side chain. Each privacy group can have its own bespoke genesis. The ramifications of this are that when you interact with a private transaction you are in effect sending two transactions in one; the public marker transaction (with it's own sender, nonce etc) and a private transaction (again with it's own sender, nonce, etc). 

To send a private transaction in besu, you must first construct a raw transaction and send that using `eea_sendRawTransaction`. This will distribute the private transaction to the participants of the privacy group. Besu uses tessera, a private p2p enclave network, to achieve this. Finally, a public transaction is submitted to the blockchain with the enclave key as the input.

Let's run through an example to explore how that looks too different parties. Say we've deployed a private contract onto a private besu network. 

First, let's take a look at the private marker transaction.

`eth.getTransaction("0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a")`

```js
{
  blockHash: "0xaf6694d5dd21cc59fed41db2e210e251342d1b817f88f2d266c65a09f053e5f2",
  blockNumber: 35,
  from: "0x21e9376e74221fd8f2ef3488cf9acde4ad44ed6f",
  gas: 3000000,
  gasPrice: 0,
  hash: "0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a",
  input: "0x4aba98105ce8b1f738ab087a655d85c61c6b9d60420e4185cc86a22678e7cc2d",
  nonce: 0,
  publicKey: "0x0fabe162accda945f5f4fe8ad3bbcb5903898e0e0cf24229764ea818308f82cc1ce2d6957a326eaa7a24cbe1c2cb70ab00647c6218a92169c892ec5e6d314833",
  r: "0xedc7ce90081f3a63716d6f1d3221e6dff3af788372b3482b24955e6088061bc0",
  raw: "0xf8808080832dc6c094000000000000000000000000000000000000007e80a04aba98105ce8b1f738ab087a655d85c61c6b9d60420e4185cc86a22678e7cc2d1ba0edc7ce90081f3a63716d6f1d3221e6dff3af788372b3482b24955e6088061bc0a009ebe6716f4b282da9e2970f1c7e4bafcbf6243f0da74a0f4ebee4c508c8fa7d",
  s: "0x9ebe6716f4b282da9e2970f1c7e4bafcbf6243f0da74a0f4ebee4c508c8fa7d",
  to: "0x000000000000000000000000000000000000007e",
  transactionIndex: 0,
  v: "0x1b",
  value: 0
}
```

Notice that the input is `0x4aba98105ce8b1f738ab087a655d85c61c6b9d60420e4185cc86a22678e7cc2d` this decodes to `SrqYEFzosfc4qwh6ZV2FxhxrnWBCDkGFzIaiJnjnzC0=` which is used to lookup the private contract in your enclave. Also notice that the to address is `0x000000000000000000000000000000000000007e` this is the address of the privacy marker pre-compile. It's what is used to indicate to besu that it's dealing with a private transaction. 

Let's take a look at the public transaction receipt:

`eth.getTransactionReceipt("0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a")`

```js
{
  blockHash: "0xaf6694d5dd21cc59fed41db2e210e251342d1b817f88f2d266c65a09f053e5f2",
  blockNumber: 35,
  contractAddress: null,
  cumulativeGasUsed: 23176,
  effectiveGasPrice: "0x0",
  from: "0x21e9376e74221fd8f2ef3488cf9acde4ad44ed6f",
  gasUsed: 23176,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: "0x000000000000000000000000000000000000007e",
  transactionHash: "0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a",
  transactionIndex: 0
}
```

Notice that everything about the transaction is private, the only information that an observer of the transaction knows are who sent it and what enclave key they used to communicate. 

Because the privacy model being used is `restricted`, only the members who have access to the contract will have it in their enclave. 

To view the private transaction receipt we can call `priv_getTransactionReceipt` with the public marker transaction hash:

```sh
curl localhost:20000 -X POST --data '{"jsonrpc":"2.0","method":"priv_getTransactionReceipt","params":["0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a"],"id":1}'
```


```js
{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : {
    "contractAddress" : "0x7634a61eb86c65130676abf837e03663df20f811",
    "from" : "0x13a52aab892e1322e8b52506276363d4754c122e",
    "output" : "0x608060405234801561001057600080fd5b50600436106100415760003560e01c80632a1afcd91461004657806360fe47b1146100645780636d4ce63c14610092575b600080fd5b61004e6100b0565b6040518082815260200191505060405180910390f35b6100906004803603602081101561007a57600080fd5b81019080803590602001909291905050506100b6565b005b61009a610115565b6040518082815260200191505060405180910390f35b60005481565b7fc9db20adedc6cf2b5d25252b101ab03e124902a73fcb12b753f3d1aaa2d8f9f53382604051808373ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019250505060405180910390a18060008190555050565b6000805490509056fea264697066735822122027c2c4639230624c056430fb1d781389b791d56f4d6fe2ca494a46a171f5433164736f6c63430007060033",
    "commitmentHash" : "0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a",
    "transactionHash" : "0xfdc9304de195fa71b9838632d38478f23391d63a8103d848eba64d5ee7d34d46",
    "privateFrom" : "BULeR8JyUWhiuuCMU/HLA0Q5pzkYT+cHII3ZKBey3Bo=",
    "privateFor" : [ "1iTZde/ndBHvzhcl7V68x44Vx7pl8nwx9LqnM/AfJUg=" ],
    "status" : "0x1",
    "logs" : [ {
      "address" : "0x7634a61eb86c65130676abf837e03663df20f811",
      "topics" : [ "0xc9db20adedc6cf2b5d25252b101ab03e124902a73fcb12b753f3d1aaa2d8f9f5" ],
      "data" : "0x00000000000000000000000013a52aab892e1322e8b52506276363d4754c122e000000000000000000000000000000000000000000000000000000000000002f",
      "blockNumber" : "0x23",
      "transactionHash" : "0xe42db8625f672c51665a7cd2275834b55a17596d5a5751ed88d3a0b92b8cdc8a",
      "transactionIndex" : "0x0",
      "blockHash" : "0xaf6694d5dd21cc59fed41db2e210e251342d1b817f88f2d266c65a09f053e5f2",
      "logIndex" : "0x0",
      "removed" : false
    } ],
    "logsBloom" : "0x00000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000020000000000010000000000000000000000000000000000002001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "blockHash" : "0xaf6694d5dd21cc59fed41db2e210e251342d1b817f88f2d266c65a09f053e5f2",
    "blockNumber" : "0x23",
    "transactionIndex" : "0x0"
  }
}
```

Notice that all members ("BULeR8JyUWhiuuCMU/HLA0Q5pzkYT+cHII3ZKBey3Bo=", "1iTZde/ndBHvzhcl7V68x44Vx7pl8nwx9LqnM/AfJUg=") of this transaction will have exactly the same result. Non-members will simply receive a null result when they call the method. 


To do anything usefull after this point we need to ascertain the `privacyGroupId` being used for these two members by calling `priv_findPrivacyGroup`

```sh
curl localhost:20004 -X POST --data '{"jsonrpc":"2.0","method":"priv_findPrivacyGroup","params":[["BULeR8JyUWhiuuCMU/HLA0Q5pzkYT+cHII3ZKBey3Bo=", "1iTZde/ndBHvzhcl7V68x44Vx7pl8nwx9LqnM/AfJUg="]],"id":1}'
```
```js
{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : [ {
    "privacyGroupId" : "kTeJS4TI7mnWhJrEItmjHe1iDepAuURPmeMVcyQOgr0=",
    "name" : "legacy",
    "description" : "Privacy groups to support the creation of groups by privateFor and privateFrom",
    "type" : "LEGACY",
    "members" : [ "1iTZde/ndBHvzhcl7V68x44Vx7pl8nwx9LqnM/AfJUg=", "BULeR8JyUWhiuuCMU/HLA0Q5pzkYT+cHII3ZKBey3Bo=" ]
  } ]
}
```

We need this for interacting with the can call the contract with `priv_call`, note this is the same as `eth_call` except you need to specify the `privacyGroupId`. In this case, you can think of the `privacyGroupId` as a side chain identifier.

```sh
curl localhost:20004 -X POST --data '{"jsonrpc":"2.0","method":"priv_call","params":["kTeJS4TI7mnWhJrEItmjHe1iDepAuURPmeMVcyQOgr0=", {"to":"0x7634a61eb86c65130676abf837e03663df20f811", "data": "0x6d4ce63c"}, "latest"],"id":1}'
```

```js
{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : "0x000000000000000000000000000000000000000000000000000000000000007b"
}
```

Note we are simply calling the `get` method on the contract `0x7634a61eb86c65130676abf837e03663df20f811` in the side chain `kTeJS4TI7mnWhJrEItmjHe1iDepAuURPmeMVcyQOgr0=`. The contract may also exist in the public or another side chain and will likely contain a different state.

# Summary

The main thing to think about with besu is that it supports private transactions that exist in their own isolated private side chain. Besu supports an unlimited number of private side chains.