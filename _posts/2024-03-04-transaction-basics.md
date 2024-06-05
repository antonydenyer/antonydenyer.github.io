---
layout: post
title: "Post EIP-1559 Ethereum Transaction Basics"
date: 2024-03-04 09:00
comments: true
categories: [transactions, ethereum]
---

There always seems to be much confusion about transactions. The introduction of EIP 1559 has somewhat confounded this; even among people in this space, you still often hear people getting things muddled. It's been almost three years, and people are still confused! This is my attempt to clarify the interplay between gas, fees, and account balance and what it means for transaction validity and likelihood of inclusion. 

The focus will be on Type 2 (EIP-1559) dynamic transactions, as this is what most wallets use now. First, some definitions:


## Gas Limit

The gas limit refers to the maximum amount of gas units the sender is willing to spend on a transaction. This limit is set by the user initiating the transaction and is included when you sign the transaction.

## Gas Used

The actual amount of gas units used when your transaction is executed at that point in time. This can change depending on the state of the blockchain. This information is not available when you sign a transaction.

## Out of Gas

When a transaction is processed, the amount of gas used accumulates over time. When processing a transaction, there can be only three outcomes. It will either succeed or fail with a revert error or an out-of-gas error. The out-of-gas error happens when the gas used exceeds the gas limit.

## Gas Price

This is the maximum price per gas you are willing to pay. With legacy Type 1 transactions, it's relatively easy to understand as it's a single value and included in your signed transaction intent as, unsurprisingly, gas price. With dynamic Type 2 (EIP-1559) transactions, there are two values to worry about max fee per gas and max priority fee per gas. They both impact the actual gas price you will pay. The gas price a transaction pays is also dynamically adjusted per block. With Type 2 transactions, the maximum fee per gas and maximum priority fee per gas is included in the signed transaction. 


# Transaction Validity

When a transaction is sent to an RPC node, the node performs some basic validity checks before it's broadcast to other peers in the p2p network. Those peers also perform similar checks. This helps keep the list of pending transactions (aka public mempool) free from junk and spam. From a gas price point of view, the main thing is whether the user has enough ETH in their wallet to cover the possible gas fees (gas limit * max fee per gas) and any value transfer they wish to perform. It's a fairly simple check, but the values entered by the user/wallet impact the transaction validity.

## Estimating the Gas Limit

Obtaining an appropriate gas limit by tricky `eth_estimateGas` is only good when estimating it. For example, if you estimated your gas at block 200 and your transaction is included at the top of block 201, then the gas estimate would be precisely correct. However, in all likelihood, your transaction will land in the middle of the block. The estimate might be slightly out because of previous state changes that were not considered as part of the estimate. 

## Setting Max Fee Per Gas

You can fetch the pending block base fee using `eth_feeHistory`. You can then sign a transaction with the pending block base fee as your max fee per gas, which will be valid for that block. However, the block base fee will almost certainly change for subsequent blocks. If the block base goes down, your transaction will still be valid; if it goes up, it will not. 

## Balance

With both values set on the transaction, you can check if the user has enough balance to attempt the transaction. The problem is that most wallets will add additional headroom for the gas limit, meaning regular users almost always overestimate it. In addition to this, you don't know if your transaction will land on-chain this block, so wallets will recommend that you sign a transaction with some multiplier of the current block base fee (on metamask, the default is 1.35). 

Let us take a look at simple [approve](https://etherscan.io/tx/0x9c88faff86425d9970a261696498d54652b19a0218d73902555cc98d6a9c1e41) in this example you can see that the gas limit set by the wallet was `68,053` ~20% higher than gas used [^1]. The max Fee Per Gas was set to `90.226598058` Gwei. The block where the transaction landed had a base fee of `66.65042672`. The balance required for the user to make this transaction was `6140190.677641074` Gwei (~$22.65[^2]). However, the transaction only used `3727159.154217342` Gwei (~$13.75). 

This means that user was required to have almost twice the amount of ETH in their wallet to transact. As the user's ETH balance reduces, the chances that the wallet will tell you you can not do something when, in fact, you can increase. 

# Priority Fees

We have yet to discuss priority fees as they further confuse the problem. Priority fees only come into play when the block base fee is higher than the transaction max fee per gas you've set. The priority fee is transferred to the block proposer as a reward for including transactions. The problem is that it's unclear what the 'right' priority fee should be to get inclusion. Let's assume that you think `1` Gwei is the correct price for priority fees to ensure your transaction lands on-chain. To have that priority fee available to the block proposer, the transaction max fee per gas must be at least 1 Gwei higher than the block base fee.

## Inclusion Latency

Simply setting a higher priority fee doesn't mean reduced latency. You need to ensure enough headroom in the max fee per gas to make the priority fee meaningful[^3]. And even then, it's at the pleasure of the block proposer as to whether you get included or not[^4]. 

# User Experience

The upshot is that users will always be signing transactions with higher values than neccessary. Users will always need to have some ETH in their wallets. Wallets should accept ETH derivatives (WETH, stETH etc) to pay for gas. That way users can park all their ETH into something yield bearing without the worry of keeping some back for gas. 


[^1]: There is some nuance with stipends and refunds where you must have a higher gas limit than gas used - but I'm ignoring that issue for this example.

[^2]: ETH price was $3688 at the time of writing

[^3]: https://www.antonydenyer.co.uk/2024-02-22-transaction-inclusion-latency/

[^4]: Many discussions are floating around the EF regarding inclusion lists and censorship resistance. https://ethresear.ch/t/no-free-lunch-a-new-inclusion-list-design/16389 