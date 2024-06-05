---
layout: post
title: "Why aren't transactions landing on chain?"
date: 2024-02-22 09:00
comments: true
categories: [block builder, mev, ethereum]
---

At MetaMask, we care about transactions getting on-chain. Historically, this hasn't always been the case. The first version of MetaMask just signed transactions; you had to provide your own RPC provider. Once the transaction was submitted, it was up to the RPC to get it on-chain. We see RPC providers offering things like reinforced transactions from Alchamy and transaction assurance from Infura. And in MetaMask, transactions are resent to the RPC provider if the transaction isn't getting included. The problem is that most users don't care about how any of this stuff works. What they do care about is why their transaction isn't getting on-chain. So we try and abstract some of that complexity away for them with things like gas parameter recommendations and basic simulations. But at the end of the day, if a transaction doesn't land, it's perceived as a problem with MetaMask. 

## Are block builders rational?
It may seem like an odd question, but it's worth considering. We've seen that relays are not profitable; Matt Curtler made the point several times before Blocknative closed its relay operations. We see multiple relays being run by parties doing it for reasons other than profit. 

Aestus, Ultrasound and Agnostic Gnosis are not making any revenue. The rest are vertically integrated block builders and will likely subsidise relay operations from elsewhere. They are probably winning more blocks due to the latency games they can play.

Are there block builders that don't do it for the profit?
A large originator of order flow may take a position in the block builder market to ensure their transactions land on-chain without interference. However, the landscape for block building is one of fierce competition. None of the dominant block-builders are selfless; they all appear to be running their block-builders to extract profit by providing infrastructure, trading activities or a combination of the two. 

## Transactions not landing
We've noticed a strange phenomenon on mainnet over the last few months. There have been many occasions where a transaction is perfectly valid, and the transaction was not included by the block builder that won the block.

We checked our assumptions to ensure it was nothing on our side.

### Block Space
The first thing we checked was block space. Is the block full? Short answer: no, the blocks where we saw this happen were not full. In most cases, they were approximately half full, so there was plenty of space left.

### Priority Fees
Did the transaction have an effective gas price? In some situations, the transaction can have zero tips, leaving no incentive for the block builder to include the transaction. While we have some instances where this happens in the overriding majority, this is not the case. 

### Transactions Valid
We ran an experiment using eth_sendBundle and titan_getBundleStats. On the off chance that we had better visibility of the mempool than a professional block builder, we submitted public transactions directly to block builders to ensure they had it in their local mempool. We did this at the beginning of the block to ensure there were no latency issues. In multiple instances, we found that transactions were not being included.

### Priority Fee Floor Price
We also checked that the priority fee we included on transactions exceeded the bottom of block transactions. Often, we would see transactions with an effective gas tip of over 0.01 gwei. Usually, this would be enough for a transaction to be included.

We were reasonably sure that it was nothing on our side. So what could it be?

### Trading Strategy Conflict
Another hypothesis was that our transactions conflicted with block builders' private order flow. But we saw this happen with simple 'approve' transactions and transfers. So it's unlikely to be that. 

### Latency Jitters
The winning strategy that the block builder is employing may be winning because of some latency jitters. You could imagine a situation where a lightweight block lands on-chain quicker than a heavy block. Or, a lightweight block can be submitted later, giving the searcher builder more time to make the most of price volatility and, therefore, creating a more valuable block by paying more.

### BaseFee Manipulation
If the block builder knows, they will send at least one transaction to the next block. It may be more profitable to forfeit the upside of the priority fee reward in return for being able to submit a cheaper transaction on the next block. 

### BaseFee Management
Alternatively, block builders may be participating in some form of base fee smoothing. You could see a situation where the next block is 150% full at the current base fee. By submitting a full block, you would increase the base fee by 12.5%, which may invalidate your remaining 50% of transactions. A more profitable strategy might be to smooth the base so that all the transactions remain valid. You would need to be confident that you would win the subsequent few blocks for this to make sense.

### Rational transaction pricing
The transactions that are not being included have low priority fees. Perhaps the block builder rationally views them as not worth the effort. If a block builder is paying $200 to win a block, do they care about collecting an additional penny? There may be situations when it makes sense to collect the pennies, which may be why we see the variance.

## Opaque Transaction Pricing
The problem is that we don't have any visibility of the price of a transaction. Some transactions are worth more than others for reasons other than the priority fee. 

### Inclusion Latency
How do you get your transactions included when you don't know how to price them? The problem is that you don't know the relative price to other transactions (that may have mev) let alone what your own transaction is worth. Predicting the priority fee you're willing to pay when the transaction inclusion is based on an opaque proprietary algorithm is impossible; users will consistently overpay for gas.

# Predictions
Adding additional priority fees will become the primary way transactions get included. When the network is busy and during times of high price volatility, history will repeat itself, and users will sign transactions with ever-increasing priority fees in the hope of getting inclusion. 

## Social Agreements
As we see more order flow being rooted privately, I suspect we will see more private agreements between parties. If a block builder doesn't include transactions sent to them when the originator feels they should, they will likely cut them off from access to that order flow. Searchers would do the same if block builders started to unbundle their bundles or stopped including their transactions. Should it be any different for 'vanilla' transactions?

## Priority Fee Floor Pricing
Another possibility is that block builders start to increase/introduce priority fee floor pricing in return for 'guaranteed' inclusion if they win the block. Larger originators could use their leverage to put downward pressure on prices and inclusion latency. 

# Summary 

You can look at the tool used to monitor public transactions; it should identify transactions that should be included in a block but are not. It's not 100% reliable, but it throws up some exciting blocks to explore. Either way more resarch is required.

[https://github.com/antonydenyer/block-builder-mempool](https://github.com/antonydenyer/block-builder-mempool)

<div style="text-align:center;">
  <a href="/assets/img/blog/transaction-inclusion-latency/web.png">
    <img src="/assets/img/blog/transaction-inclusion-latency/web.png" alt="web table of missing transactions">
  </a>
</div>

