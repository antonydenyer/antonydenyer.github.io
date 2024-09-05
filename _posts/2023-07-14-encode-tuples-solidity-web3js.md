---
layout: post
title: "How to encode tuples using web3js for multicall3"
date: 2023-07-14 09:00
comments: true
categories: [ethereum, solidity, web3js, simulation]
---

We're going to be looking at sending multiple transactions in a single request using [multicall3](https://github.com/mds1/multicall)

The function `aggregate3` takes an array of `Calls3` to encode the request can be a little bit tricky. 

```solidity
struct Call3 {
    // Target contract to call.
    address target;
    // If false, the entire call will revert if the call fails.
    bool allowFailure;
    // Data to call on the target contract.
    bytes callData;
}

struct Result {
    // True if the call succeeded, false otherwise.
    bool success;
    // Return data if the call succeeded, or revert data if the call reverted.
    bytes returnData;
}

/// @notice Aggregate calls, ensuring each returns success if required
/// @param calls An array of Call3 structs
/// @return returnData An array of Result structs
function aggregate3(Call3[] calldata calls) public payable returns (Result[] memory returnData);
```

First, we will encode the functions we want to perform in the multicall. 

```js
const hello = web3.eth.abi.encodeFunctionCall({ 
    name: 'setGreeting',
    type: 'function',
    inputs: [{
        type: 'string',
        name: '_greeting'
    }]
}, ['bonjour'])

const goodbye = web3.eth.abi.encodeFunctionCall({ 
    name: 'setGreeting',
    type: 'function',
    inputs: [{
        type: 'string',
        name: '_greeting'
}]
}, ['au revoir'])

const greet = web3.eth.abi.encodeFunctionCall({
    name: 'greet',
    type: 'function'
})

```

We're then going to encode our call to the multicall contract. First, we need to put together our abi data.

```js

const abi = {
	"stateMutability": "payable",
	"type": "function",
	"name": "aggregate3",
	"inputs": [{
		"components": [{
				"internalType": "address",
				"name": "target",
				"type": "address"
			},
			{
				"internalType": "bool",
				"name": "allowFailure",
				"type": "bool"
			},
			{
				"internalType": "bytes",
				"name": "callData",
				"type": "bytes"
			}
		],
		"internalType": "struct Multicall3.Call3[]",
		"name": "calls",
		"type": "tuple[]"
	}],
	"outputs": [{
		"components": [{
				"internalType": "bool",
				"name": "success",
				"type": "bool"
			},
			{
				"internalType": "bytes",
				"name": "returnData",
				"type": "bytes"
			}
		],
		"internalType": "struct Multicall3.Result[]",
		"name": "returnData",
		"type": "tuple[]"
	}]
}
```

Now we can put together our multicall using web3js

```js

const multiCall = web3.eth.abi.encodeFunctionCall(abi, [[
    { 'target': '0x8d9fBC02598e32C8d594bbEF4257846653ff4732', 'allowFailure': false, "callData": hello },
    { 'target': '0x8d9fBC02598e32C8d594bbEF4257846653ff4732', 'allowFailure': false, "callData": greet },
    { 'target': '0x8d9fBC02598e32C8d594bbEF4257846653ff4732', 'allowFailure': false, "callData": goodbye },
    { 'target': '0x8d9fBC02598e32C8d594bbEF4257846653ff4732', 'allowFailure': false, "callData": greet },
]])

```

Note the encodeFunctionCall takes an array of parameters. In our case, the first parameter of the function call is an array of type `Call3`. To correctly encode the function call, the property names must match abi name. 

Now we have encoded our multicall function, we can go ahead and simulate the transaction with `eth_call`

```js
 const hexString = await web3.eth.call({
      from: '0x673D2EBe4B6BAA946345C7b1F8d3Cc2FfB3429Bf',
      to: '0xa252BaE8352949C85606d77D3E1d44B71a202a32',
      data: multiCall,
  })

```

Then we need to decode the response again.

```js
const multiCallResult = web3.eth.abi.decodeParameters(abi.outputs, hexString).returnData
```

Et Voila! 


Full gist [available here](https://gist.github.com/antonydenyer/9f58b81f6fe2ede4baa1c02d1c71d741)