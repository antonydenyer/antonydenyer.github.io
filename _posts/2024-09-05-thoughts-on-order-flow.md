---
layout: post
title: "Thoughts on Order Flow"
date: 2024-09-04 09:00 +0000
comments: true
categories: [ethereum, mev, mempool, block builders]
---

In recent weeks, [EigenPhi](https://eigenphi.substack.com/p/private-order-flows-the-sleeve-bidding) and [Blocknative](https://www.blocknative.com/blog/ethereum-private-transactions-the-flippening) have shone a light on a fact that's been quietly shaping the landscape of block building: the majority of block builder revenue comes from private order flow. While some may find this surprising, it's a logical outcome when you dive deeper into the mechanics at play.

tldr; The economics of block building are far more complex and competitive than they might first appear. In the long run validator priority fees and block builder profits will trend to zero as value extraction happens further up the supply chain.

## The Nature of Public Mempool Data

Public mempool data is, by its very definition, accessible to everyone—block builders, validators, and any other interested parties. This universality creates a common knowledge scenario where all participants can see the same public transactions. Although there are edge cases involving propagation delays, the general assumption is that block builders and validators see the same transactions.

Now, consider the role of the proposer. A rational proposer won't choose a block that's less valuable than one they could assemble locally. Consensus clients like Teku enforce a default where the block builder's submission must exceed the local block value by at least 10% to be considered. This dynamic naturally pushes block builders to hand over the priority fees from public transactions to the proposer.

### The Profit Equation for Block Builders

Given this context, how does a block builder generate profit? It's certainly not by creating more valuable blocks from the same set of public transactions. Selecting transactions with the highest priority fees isn't rocket science; implementations are pretty straightforward. There might be some edge cases when blocks are full, but fundamentally, public order flow isn't a money-maker.

Block builders' real profits come from exclusive access to order flow that isn't available to competitors, be they other block builders or proposers. This exclusivity is the key to extracting value in a highly competitive environment.

### The Role of Transaction Originators

Why do transactions include priority fees? In theory, they incentivise the proposer to maximise block space utilisation. The proposer gets paid to pack as many transactions as possible into each block. In theory, a higher priority fee should mean faster inclusion. Searchers are competing for a position in the block and are sophisticated enough to reason about this. Everyone else just wants timely inclusion; in reality, this is the minimum floor price for the priority fee. Put another way, if a retail user submits a transaction with 1 gwei tip vs 2 gwei tip it probably won't make any difference whatsoever to their incluion latency.

### Block Builder Profit and The Flow of Priority Fees

So, how do block builders make money in such a competitive environment? Consider a searcher with a low-value opportunity only viable in the upcoming block. If the transaction is sent to just one block builder, there's no guarantee they'll win the next block, so the searcher sends it to multiple builders. What happens next? The builders end up bidding away the priority fees to the proposer because they know their competitors will do the same. In this scenario, whichever block builder wins walks away with no profit.

To make money, block builders need exclusive deals, they will offer sweeteners that set them apart from the competition. But even then,  relentless competition will drive profits down. Block building is, and should be, a race to the bottom, so it's unlikely to remain excessively profitable in the long run.

## Cui Bono?

So, where do the priority fees end up? It depends on the transaction supply chain and what is upstream from the block builder. 

### Searchers
The searcher may have some exclusive arbitrage that only they know about or can execute. Will they bid up priority fees to get their transaction on the chain? I doubt it; they will pay the minimum to get it on-chain safely. They will not bid away their profits to the proposer.

### Retail Order Flow
There are some nuances here that I think require further clarification:

RPC Provider - the entity that responds to RPC requests, e.g. `eth_call`.

Transaction Gateway - the thing that handles requests `eth_sendRawTransaction` and decides how to route it. There is a similar subtle difference between a node provider and an RPC provider.

Standard Transactions - the ones that you sign and send using `eth_sendRawTransaction`. 

Meta Transaction -  anything that leverages off-chain signatures and potentially involves a relayer to execute actions on-chain.

#### User Intents 
In the case of meta transactions, the priority fees usually are 'free' and consist of some form of limit order. The user will pay for transaction fees in the swap spread or another way (perhaps not being filled or with slow execution, etc). Depending on the protocol, the searcher/filler will pay to get the transaction on-chain. If the meta transaction is made available publically, the profit from the spread will be bid away and will end up in the hands of the proposer. Or if it's kept exclusive, the searcher will keep it. If the mechanism design is done right, you will end up with better prices for the user. There's potential for gas savings with batch transactions and other optimisations. If done right, this is great for users but bad for proposers - priority fees will trend downwards. 

### Transaction Gateways
I'm separating these from what people call RPC providers because when you request information from an RPC provider, it's the same no matter which one you use. However, the behaviour can vary wildly when you send transactions with `eth_sendRawTransaction`. For example, a standard node will send the transaction to its peers. Alchemy and Infura attempt to rebroadcast transactions that they think have been dropped. Bloxeroute attempts to propagate transactions aggressively using its proprietary BDN. They are all deviations from standard behaviour and have different price points.

The transaction gateway can extract priority fees for every transaction it receives. However, it can only do this if it retains exclusivity. What the transaction gateway does with those fees is up to them. It will also depend on its relationship with customers and users. For example, with CowSwap, some of the "surplus" goes to the user, and with flashbots, you get "refunds". 

The transaction gateway also expresses its preference for execution speed. By retaining exclusivity over transactions for longer periods, the gateway could exercise collective bargaining power by accumulating and aggregating many transactions. Or it could opt for faster inclusion by giving away more priority fees. Transaction gateways with larger pools will benefit from economies of scale as they will be able to leverage their size with block builders. They can drive down priority fees and improve user execution speeds. 

### The block builders of tomorrow
As dApps and wallets attempt to retain exclusivity over their order flow, they will naturally tend towards being pseudo-block builders. Transaction gateways, will be able to fill user intents using different combinations of tactics and will be able to leverage their order flow to combine and optimize routing. They will be able to drive down gas consumption by optimally combining transactions from various sources. Block builders do not do this currently as there's no upside for them. 


## Conclusion
As the ecosystem evolves, the race to the bottom in block-building profits will likely intensify. The winners will be those who can most effectively leverage exclusive order flow, while users stand to benefit from the efficiencies and innovations that arise in this competitive landscape.

